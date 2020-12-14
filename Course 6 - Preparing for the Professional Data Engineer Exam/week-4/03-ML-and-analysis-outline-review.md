# ML and Analysing - outline review

## Machine Learning and Analysis

### Machine Learning

| Topic | Tips |
|-------|------|
| **Feature selection/engineering** | The process of selecting relevant predictors for building a model |
| **Algorithm selection** | Google ML uses general algorithms that are applicable to a wide variety of datasets. Categories: Linear and DNN, Regressor and Classifier |
| **Debugging a model** | Code, data, features, errors,. Look for outliers and verify features |

### Analysing data

| Topic | Tips |
|-------|------|
| **Data collection and labelling** | What labels are required for you application and/or for training ML? Can the data be labelled as it is being collected? If not, what strategy will you use for labelling data?
| **Data visualisation** | Make complex data easier to understand by surfacing trends, outliers and patterns |
| **Dimensionality reduction** | Reduce random variables being considered, focus on meaningful variables. Feature selection and feature reduction |
| **Data cleaning/normalisation** | Normalising/de-normalising is a tradeoff between efficiency and performance |
| **Define success metrics** | Assumed perfection, 100%, often is unrealistic. How will you know that the results are acceptable unless you define a measure or condition of success? |

### Machine learning model deployment

| Topic | Tips |
|-------|------|
| **Performance/cost optimisation** | Testing and tuning goes a long way to making your model practical and cost effective. How to get from "toy" solution to "production" solution? |
| **Online/dynamic learning** | Dynamic learning improves the model over time. Often the data is already available. Consider the development of a model to be an ongoing process. The model will not be generated one time, it will be iterative. The model will need to be updated and you will want the model to improve |

#### Exam Tip #1

> **Practical, not perfect!**
>
> This is key to avoiding runaway costs and performance problems

### Recovering data

| Topic | Tips |
|-------|------|
| **Planning (e.g. fault-tolerance)** | Persist datasets / pcollections|
| **Executing (e.g., re-running failed jobs, performing retrospective re-analysis** | Persist |
| **Stress testing data recovery plans and processes** | Measure / manage "error budget". Make recovery part of regular activities, so you kow they are working |

#### Exam Tip #2

> **Make data recovery a part of your regular routine**
>
> When it comes to backup and restore, people care about restoring the data
> How would you know if your recovery system is working if you don't periodically test it?

