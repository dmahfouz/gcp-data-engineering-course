# BigQuery ML

BigQuery Machine Learning allows you to build machine learning models using SQL syntax. This is a really powerful feature since anyone who knows only SQL can build ML models.

BigQuery ML brings us one step closer to democratising Machine Learning and AI, one of the core missions of Google.

## Agenda

- BigQuery ML for Quick Model Building
  - First we'll introduce BigQuery ML and walk through the steps for an end-to-end use-case  
- Classification, Regression and Recommender Models
  - Then, we'll discuss the current zoo of support and models  
- Unsupervised ML with Clustering Models
  - At the end, we will go through some hands-on labs using BQML

## BigQuery ML for Quick Model Building

### BigQuery ML is a way to build custom models

- Where does BQML fit in the greater picture of Google Cloud's AI and ML options?
- Unlike ML APIs, you are able to create your own custom models
- With AutoML users can leverage Google's ML models in certain cases to build their own models from scratch using transfer learning and a form of neural architecture search
- With BQML, the process of training, evaluating and scoring a model can be done easily in a short amount of time using BigQuery's SQL syntax

![bq-eco](./imgs/bq-eco.jpeg)

### Working with BigQuery ML

- To work with BQML, there are only a few steps from beginning to inference:

  1. **Dataset**: first, we must write a query on data stored in BigQuery to extract our training data
  2. **Create/train**: then, we can create a model where we specify a model type and other hyperparameters
  3. **Evaluate**: after the model is trained, we can evaluate the model and verify that it meets our requirements
  4. **Predict/classify**: finally, we can make predictions using our model on data extracted from BigQuery

## Classification, Regression and Recommender Models

## Unsupervised ML with Clustering Models
