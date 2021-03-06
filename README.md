# Optimizing a ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary
**Problem Statement and Data**

The data is related to the direct marketing campaigns of a Portuguese banking institution. The marketing campaigns were based on phone calls. Often, more than one contact to the same client was required, in order to access if the product (bank term deposit) would be ('yes') or not ('no') subscribed.
i.e. The classification goal is to predict if the client will subscribe (yes/no) to a term deposit (variable y).

**Workflow**

Hyperparameters are adjustable parameters you choose for model training that guide the training process. To solve the above problem we use two approaches of automated hyperparameter tuning for our machine learning model.
1) HyperDrive - The HyperDrive package helps you automate choosing these parameters for a given model.
2) AutoML - AutoML or Automated ML, is the process of automating the task of machine learning model development. Using this feature, you can predict the best ML model, and it's hyperparameters suited for your problem statement.

This is the workflow followed in the experiment.
![alt text](https://video.udacity-data.com/topher/2020/September/5f639574_creating-and-optimizing-an-ml-pipeline/creating-and-optimizing-an-ml-pipeline.png)

## Scikit-learn Pipeline
**Classification algorithm, Pipeline architecture and Hyperparameter tuning**

Logistic Regression is a Machine Learning algorithm that is used to predict the binary probability of a categorical dependent variable.
In this section, we use HyperDrive on the Logistic Regression model imported from the sklearn library.

The HyperDrive configuration here includes data about hyperparameter space: 
- Sampling (RandomParameterSampling, GridParameterSampling, BayesianParameterSampling) 
- Termination policy (BanditPolicy, MedianStoppingPolicy, TruncationSelectionPolicy)
- Primary metric
- Estimator
- Compute Target to execute the experiment runs on
These configurations are passed on for Hyper Parameter tuning.

**Parameter sampling**

In this experiment, the parameter sampler used is Random Sampling. 
In random sampling, hyperparameter values are randomly selected from among the given search space.
Random sampling is chosen because it supports discrete and continuous hyperparameters, and early termination of low-performance runs.

The hyperparameters to be optimized in the Logistic Regression algorithm are --C and --max_iter. 
Here C value is the Inverse of regularization strength(smaller values specify) stronger regularization. A continuous range of values is provided for the Random Sampler to choose from.
max_iter is the maximum number of iterations it takes for the optimization algorithm to converge. A list of equally spaced options for Max Iterations is provided.
From this combination of hyperparameter options, HyperDrive randomly chooses one for each to then find hyperparameter leading to optimal results.

**Early stopping policy**

In this experiment, the early stopping policy used is Bandit Policy. Bandit policy is based on the difference in performance from the current best run, called 'slack'. Here the runs terminate where the primary metric is not within the specified slack factor/slack amount compared to the best performing run. 
Slack factor gives the allowed slack as a ratio. Slack amount specifies the allowable slack as an absolute amount, instead of a ratio. Other optional configurations include evaluation_interval which is the frequency for applying the policy, and delay_evaluation which delays the first policy evaluation for a specified number of intervals

Here any training runs whose best metric defined at the interval is less than "best_metric/(1+slack_factor)" will be terminated. Bandit policy filters out all runs, with sub-optimal performance.

After running the SkLearn pipeline, the best performing hyperparameters are --C as 6.429755013675407 and max_iter as 250, with an accuracy of 0.9133535660091047.


![image](https://user-images.githubusercontent.com/59551550/103478556-91954980-4ded-11eb-8321-6a08749ebd74.png)

The HyperParameters chosen are 
![image](https://user-images.githubusercontent.com/59551550/103473508-6431a700-4dbf-11eb-83ba-8f4b4c821250.png)


## AutoML
**AutoML Output**

For training, Azure ML creates multiple pipelines in parallel which try on different algorithms and parameters. It iterates through ML algorithms paired with feature selections, and each iteration produces a model with a training score. The higher the score, the better the fitting the model is. The process will stop once the exit criteria defined is hit.

After running the AutoML pipeline, the best performing model is found to be VotingEnsemble with an Accuracy value of 0.9171167420803277.
VotingEnsemble combines conceptually different machine learning classifiers and uses a majority vote or the average predicted probabilities (soft vote) to predict the class labels. This is method balances out the individual weaknesses of the considered classifiers.

**Voting Classifier**

The AutoML Voting Classifier is made up of a combination of xgboostclassifier models and lightgbmclassifier with different hyperparameter values and normalization/scaling techinques. There are 7 estimators used in the run with weights equal to 0.13333333333333333, 0.2, 0.13333333333333333, 0.06666666666666667, 0.2, 0.13333333333333333, 0.13333333333333333 respectively.

Considering the lightgbmclassifier, it is the model with one of the highest weights (20%) of the ensemble, 
The hyperparameters generated for the model are: 
'boosting_type': 'gbdt'
'class_weight': None
'colsample_bytree': 1.0
'importance_type': 'split'
'learning_rate': 0.1
'max_depth': -1
'min_child_samples': 20
'min_child_weight': 0.001
'min_split_gain': 0.0
'n_estimators': 100
'n_jobs': 1
'num_leaves': 31
'objective': None
'random_state': None
'reg_alpha': 0.0
'reg_lambda': 0.0
'silent': True
'subsample': 1.0
'subsample_for_bin': 200000
'subsample_freq': 0
'verbose': -10


The results of the run are as follows
![image](https://user-images.githubusercontent.com/59551550/103473514-6b58b500-4dbf-11eb-915d-eaeebe37e5cd.png)

Confusion Matrix of AutoML output
![image](https://user-images.githubusercontent.com/59551550/103473516-714e9600-4dbf-11eb-8030-6227263cd8d9.png)


## Pipeline comparison
The sklearn model optimized using HpyerDrive and the AutoML models are both run on the same data.

**Model Accuracy**
HyperDrive provided an accuracy of 0.9133535660091047 while the AutoML gave a better accuracy of 0.9171167420803277.

**Algorithm**
HyperDrive optimization has to be preconfigured with the necessary model, if the model is unknown or ill-suited for the data, then there is a chance of low accuracy. AutoML selects the best model and hyperparameters from a range of available algorithms. It aims to give the best accuracy among all combinations.

**Time Consumption**
Since HyperDrive already has the algorithm to use defined, the run takes very little time. AutoML which needs to be thorough takes much longer to execute.

**Feature engineering**
HyperDrive does not provide in-built featurization, feature engineering for the data has to be done as a preprocessing step. In AutoMl featurization is applied to the experiments automatically, it also provides facilities to customize featurization too.

## Future work
Future work on the experiment can include

**Model selection** Select different ML algorithms to apply hyperdrive upon. This could lead to more precise models.

**Sampling the hyperparameter space** Other parameter sampling methods to use over the hyperparameter space could be implemented i.e. Grid sampling, Bayesian sampling.

**Early termination policy** Azure Machine Learning supports the following early termination policies other than Bandit policy: Median stopping policy, Truncation selection policy. Different Early termination policies could be applied to keep the run most time/cost-efficient, yet having the best results. 

**Resource allocation** Different resource allocation in terms of max_total_runs, max_duration_minutes  or max_concurrent_runs for HyperDrive configuration , and different  experiment_timeout_minutes for AutoML can be attemoted.

**Cross-Validation** Change the number of cross-validation folds in the AutoML run.

**Primary metric** For the given data, the dataset is pretty imbalanced in terms of classes. A primary metric other than accuracy would be more apt for the given scenario i.e. AUC, Precision, Recall.


