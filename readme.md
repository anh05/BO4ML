[![Actions Status](https://api.travis-ci.org/anh05/BO4ML.svg?branch=master)](http://travis-ci.org/anh05/BO4ML)
# BO4AutoML: Bayesian Optimization library Automated machine learning 

Copyright (C) 2019-2021 [ECOLE Project](https://ecole-itn.eu/), [NACO Group](https://naco.liacs.nl/)

### Contact us

Duc Anh Nguyen

Email:d-dot-a-dot-nguyen-at-liacs-dot-leidenuniv-dot-nl

Website: [ecole-itn.eu](https://ecole-itn.eu/)
## Installation
### Requirements

As requirements  mentioned in `requirements.txt`, [hyperopt](https://github.com/hyperopt/hyperopt) as build dependencies:

```shell
pip install hyperopt
```
### Installation

You could either install the stable version on `pypi`:

```shell
pip install BO4ML
```

Or, take the lastest version from github:

```shell
pip install git+https://github.com/ECOLE-ITN/BO4ML.git
```
--
```shell
git clone https://github.com/ECOLE-ITN/BO4ML.git
cd BO4ML && python setup.py install --user
```

## Example
Define a Seach space
```python
from BanditOpt.BO4ML import ConfigSpace, ConditionalSpace, AlgorithmChoice, CategoricalParam, IntegerParam, FloatParam, Forbidden
search_space = ConfigSpace()
# Define Search Space
#1st Operator: Resampling technique
smo_type = AlgorithmChoice([['NO'], ['SMOTE', 'BorderlineSMOTE']
            , ['SMOTEENN', 'SMOTETomek'],['NearMiss', 'TomekLinks']], 'resampler')
#2nd Operator: Classifier

alg_namestr = AlgorithmChoice(["SVM", "RF"], "alg_namestr")
# Define Search Space for Support Vector Machine
kernel = CategoricalParam(["linear", "rbf", "poly", "sigmoid"], "kernel")
C = FloatParam([1e-2, 100], "C")
degree = IntegerParam([1, 5], 'degree')
coef0 = FloatParam([0.0, 10.0], 'coef0')
gamma = FloatParam([0, 20], 'gamma')
# Define Search Space for Random Forest
n_estimators = IntegerParam([5, 100], "n_estimators")
criterion = CategoricalParam(["gini", "entropy"], "criterion")
max_depth = IntegerParam([10, 200], "max_depth")
max_features = CategoricalParam(['auto', 'sqrt', 'log2'], "max_features")
# Add Search space to Configuraion Space
search_space.add_multiparameter([smo_type,alg_namestr, kernel, C, degree, coef0, gamma
                                    , n_estimators, criterion, max_depth, max_features])
# Define conditional Space
con = ConditionalSpace("conditional")
con.addMutilConditional([kernel, C, degree, coef0, gamma], alg_namestr, "SVM")
con.addMutilConditional([n_estimators, criterion, max_depth, max_features], alg_namestr, ["RF"])
# Define infeasible space (if any)
#forb = Forbidden()
#forb.addForbidden(abc,["A","C","D"],alg_namestr,"SVM")
```
Load iris data
```python
from sklearn import datasets
iris = datasets.load_iris()
X = iris.data
y = iris.target
```
Define an objective function which returns a real-value
```python
from sklearn.svm import SVC
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier
import numpy as np
from imblearn.over_sampling import SMOTE, BorderlineSMOTE
from imblearn.under_sampling import NearMiss, TomekLinks
from imblearn.combine import SMOTEENN, SMOTETomek
from sklearn.metrics import accuracy_score

def obj_func(params):
    global X,y
    params = {k: params[k] for k in params if params[k]}
    resampler = params.pop('resampler')
    if (p_sub_type == 'SMOTE'):
        smo = SMOTE()
    elif (p_sub_type == 'BorderlineSMOTE'):
        smo = BorderlineSMOTE()
    elif (p_sub_type == 'BorderlineSMOTE'):
        smo = BorderlineSMOTE()
    elif (p_sub_type == 'SMOTEENN'):
        smo = SMOTEENN()
    elif (p_sub_type == 'SMOTETomek'):
        smo = SMOTETomek()
    elif (p_sub_type == 'NearMiss'):
        smo = NearMiss()
    elif (p_sub_type == 'TomekLinks'):
        smo = TomekLinks()

    classifier = params['alg_namestr']
    params.pop("alg_namestr", None)    
    if (classifier == 'SVM'):
        clf = SVC(**params)
    elif (classifier == 'RF'):
        clf = RandomForestClassifier(**params)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y)
    if (resampler== "NO"):
        X_smo_train, y_smo_train=X_train, y_train
    else:
        X_smo_train, y_smo_train=smo.fit_resample(X_train, y_train)
    y_test_pred = clf.fit(X_smo_train, y_smo_train).predict(X_test)
    score = accuracy_score(y_test,y_test_pred)
    return -score

```
Optimizing ...
```python

from BanditOpt.BO4ML import BO4ML
opt = BO4ML(search_space, new_obj, 
            conditional=con, #conditional 
            #forbidden=forb, #No infeasible space defined in this example
            HPOopitmizer='hyperopt', #use hyperopt
            max_eval=100, #number of evaluations
            n_init_sample=50, #number of init sample 
            hpo_algo="tpe", #tpe, rand, atpe, anneal
            SearchType="full"# set "full" to use our sampling approach. Otherwise, the original library to be used
            )
best_param, min_value, listofTrial, eval_count = opt.run()
print(best_param, min_value)
#listofTrial: see hyperopt document for ``trails''
```
## Cite

Duc Anh Nguyen, Anna V. Kononova, Stefan Menzel, Bernhard Sendhoff and Thomas Bäck. Efficient AutoML via Combinational Sampling. IEEE Symposium Series on Computational Intelligence (IEEE SSCI 2021)

## Acknowledgment

This project has received funding from the European Union’s Horizon 2020 research and innovation programme under grant agreement No 766186 (ECOLE).