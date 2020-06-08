# Analytics and AI

## What is ML?

### Defining machine learning

> ML is a way to use standard algorithms to derive predictive insights from data and make repeated decisions

![what-is-ml](./images/what-is-ml.jfif)

- To derive predictive insights from data and make repeated decisions, you do this using algorithms that are relatively general and applicable to a wide variety of datasets
- Think of a company and how they use their data today:
  - Perhaps they have a dashboard that business analysts and decision-makers view on a daily basis, or report that's read on a monthly basis
  - This is an example of a backward-looking use of data - looking at historical data to create reports and dashboards
  - This is what people tend refer to when they talk about **Business Intelligence (BI)**
  - A lot of data analytics including BI is backward-looking
- To generate forward-looking or predictive insights we use **Machine Learning (ML)**
  - By looking at historical data, we can be able to make decisions based off forward-looking or predictive insights
  - Although some business analysts are able to do this - examine the data and suggest new policies or rules, for example: if it's possible to raise the price of a product in a certain region
  - They may face issues with making predictive insights without ML such as:
    - Are those predictive insights scalable?
    - Can the business analyst make such a decision for every single product in every single region?
    - Can they dynamically adjust the price every second?

### Takeaway

- In order to make decisions around predictive insights repeatable, you need ML
- ML is about making (many) predictive insights from data at a time
- It's about scaling up BI and decision-making

### ML uses standard algorithms

- The other part of the ML definition is around the use of standard algorithms
- ML uses standard algorithms to solve what looks like seemingly different problems
- Normally when we think of software, we think that programs do different things
- For example, software that you use to file your taxes is very different from the software that is used to get directions home when you're driving
- With ML, it is different as you may use the same algorithms under the hood - this is what is meant when we say ML uses standard algorithms
- But these algorithms/software can be trained to do very different things
- You can train the software to estimate the amount of taxes that you owe, or train the same software to estimate the amount of time it will take to get home

### Model training requires examples

- We use the term model because it's an approximation, it's a *model* of reality
- For example, when giving the computer lot's of historical data on drive times in New York City, the model learns relationships in the data such as traffic patterns, seasonality, time of day impact, to predict today's commute time home
- Whatever the domain ML modelling requires lots of training examples
- We can train a model to estimate tax by showing it many examples of prior year tax returns
- Or we can train a model to estimate trip duration by showing it many different journeys
- So, **the first stage of ML is to train a ML model with lots of good examples**

### Training an ML model with examples

![ml-lifecycle](./images/ml-lifecycle.jfif)

- An example consists of an input and the correct answer for that output, that's called the **Label**
- In the case of structured data that is rows and columns of data, an input can simply be a single row of data
- In unstructured data like images, an input could be a single image say a cloud that you want to classify - is this a rain cloud or not
- Now, imagine you work for a manufacturing company, you may want to train a ML model to detect defects in the parts before they get assembled into final products for users
- Some of these parts may be good and some of the parts could be fractured or broken up
- And for each image, you'll assign the corresponding label
- And then use this set of examples as training data for your model

### Predict with a trained model

![predict](./images/ml-predict.jfif)

- After you train your model, you can then use it to predict the label of images that it has never seen before


