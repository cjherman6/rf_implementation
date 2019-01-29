# Using a Random Forest for Price Prediction and EDA

## The Data
This project analyzed data on bulldozer prices and included 53 predictors on a given bulldozer including the year, sale date, enclosure type (open, enclosed, enclosed w/ AC), etc.

![Bull Dozer Data](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/data.png)

## Cleaning the Data

The dataset includes a lot of categorical variables in the form of strings, as well as variables that have a lot of missing values.

I don't want to delete any of the data so early in the project, so I used a function that will convert all string values into categories and used their category codes to feed my data into the model.

The next thing I had to address was continuous variables that had missing values.  What I did is fill the missing values with the columns median, but also created a new column that indicates whether that row had a null value for that column. This way the model can identify if a missing value is important for this variable.

_(ex: a NaN value for MachineHoursCurrentMeter might mean it's new and hasn't been used, this could be important)_

## The Model

For this project I used a **random forest** because of its flexibility with different datatypes and data relationships, as well as its ability to provide insight using feature importance, partial dependence plots, and prediction interpretations using ELI5 (this is what we'll be covering).

_A random forest uses decision trees which split on the feature that gives the highest information gain on a models predictions, and continues doing so until each data point has its own leaf (unless its configured to stop sooner)._

![Decision Tree](http://engineering.pivotal.io/images/interpreting-decision-trees-and-random-forests/multi_clf_dt_path.png)

_A random forest uses multiple decision trees that take subsamples of the dataset and split on a subset of the data features.  Once the estimators spit out their predictions, the mean of all of these predictions is what the random forest will use to predict price._

![Random Forest](https://databricks.com/wp-content/uploads/2015/01/Ensemble-example.png)

## Evaluating Features

_Feature Importance is calculated by randomizing a certain variable in your data (e.g. Randomizing YearMade), passing it through the trained model, and calculating how the prediction accuracy changes when this variable is randomized_

Using Feature importance, we can see that when YearMade and ProductSize were randomized, there were significant drops in model accuracy; meaning these features are very important in predicting price:

![Initial Feature Importance](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/feature_importance.png)


### Trimming Unimportant Features:

At this point you have a decent idea of what features are important to predict a bull dozers price.

A good way to trim out non-predictive features is to 1) check the accuracy of the model, 2) drop any values below a certain feature importance threshold (e.g. < 0.005), and 3) check the accuracy of your model again to see if any of the dropped values impact your models score.

![Feature Importance after Dropping Values](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/feature_importance2.png)

**After keeping only the features above, the validation score went from .9001 to .9033**

### Removing Redundant Features:

Another great insight is to use a correlation matrix along with a dendrogram to identify any redundant features:

![Feature Importance after Dropping Values](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/dendrogram.png)

We can see that there are a lot of variables that are very closely related (e.g. saleYear & saleElapsed, Grouser_Tracks & Coupler_System, etc.).  We can use a similar method that we used in feature importance and test to see the models accuracy when these features are removed.

**We are now at a point where we have reduced the models dimensionality without sacrificing any accuracy.**

## Insights

### One Hot Encoding

Not only is it important to look at the feature importance through a categorical lense, but you can also find insights by breaking apart certain columns using one hot encoding.

For this section, any feature that had 7 unique values or less, I used pd.get_dummies to see if any specific feature values are predictive of price:

![Feature Importance after One Hot Encoding](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/one_hot_encoding.png)

_A category in Enclosure has shown to be extremely important with one hot encoding, this variable is telling us that if a bulldozer has an enclosed cockpit with AC, the salesprice is predicted to increase_

### Partial dependence

It's great to know what features are important to a models prediction (i.e. a bulldozers expected price).  However, it is also important to know **how** a feature affects a models prediction.  One way to do this is prediction interpretation (which I'll explain next), but another way is through calculating partial dependence, which shows how a prediction is affected holding all other variables constant.

#### Enclosure (Enclosure EROPS w AC):

![Enclosure Partial Dependence](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/pdp2.png)

When a bulldozer is enclosed with an air conditioning unit, this is predictive of a higher sale price.

#### Year Made:

![Year Made Partial Dependence](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/pdp1.png)

The **newer** the model, the higher the sale price of a bull dozer.

#### Product Size:

![Product Size Partial Dependence](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/pdp3.png)

The **bigger** the bull dozer, the higher its sale price.

_Partial Dependence is calculated in a similar manner to feature importance, but instead of randomizing the variable, you hold that variable constant and see how each change in that variable effects predictions (e.g. changing all rows to be the year 1990 and then comparing that to all rows being 2004)_

### Prediction Explanation:

Another way to see how our features can affect a bull dozers sale price is using ELI5 to explain what features contributed to a specific prediction (i.e. an individual row):

A good explanation of how this works can be found [**here**](http://blog.datadive.net/interpreting-random-forests/).

_In a nutshell: These explanations are done by calculating the change in a given measurement (e.g. average sale price) between nodes after splitting_

![Tree Interpreter](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/tree_interpreter.png)

###Examples:

#### This is a large, 2007 bull dozer that comes with AC, it's sale price was $105,000
![Prediction Interpretation](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/large_2007_wAC.png)

#### This is a mini, 1998 bull dozer that has no AC, it's sale price was $
![Prediction Interpretation](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/mini_1998_woAC.png)

#### This is a mini, 1988 bull dozer that has no AC, it's sale price was $25,000
![Prediction Interpretation](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/mini_1988_woAC.png)

These further confirm what we saw earlier, that size, year, and AC are good predictors of a bulldozers sale price.

![Sale Medians](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/summary.png)

## Conclusion:

The model finished with a validation accuracy of .91 and we were able to identify features that were important in predicting sale price (YearMade, ProductSize, Enclosure).  This is a framework that can be used across any tabular data to find insights on what features are important in predicting a given dependent variable.
