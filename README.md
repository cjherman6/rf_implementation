# Using a Random Forest for Price Prediction and EDA

## The Data
This project analyzed data on bulldozer prices and included 53 predictors on a given bulldozer including the year, sale date, enclosure type (open, enclosed, enclosed w/ AC, etc.)

![Bull Dozer Data](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/data.png)

## Cleaning the Data

The dataset includes a lot of categorical variables in the form of strings, as well as variables that have a lot of missing values.

I don't want to delete any of the data so early in the project, so I used a function that will convert all string values into categories and use the category codes to feed into the model.

The next thing I did with continuous predictors that had missing values is fill the missing values with the columns median, but also create a new column that indicates whether that row had a null value or not, so the random forest can see if a missing value is significant.

## The Model

For this project I used a **random forest** because of its flexibility to different datatypes and patterns, as well as its ability to provide insight using feature importance, partial dependence plots, and prediction interpretations using ELI5.

## Insights

Using Feature importance you can see that the year of the bulldozer is very predictive of its price:

![Initial Feature Importance](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/feature_importance.png)

### Trimming Unimportant Features:

At this point you have a decent idea of what features are important to predict a bull dozers price, a good way to trim out non-predictive features is to check the accuracy of the model, drop any values below a certain feature importance threshold (e.g. < 0.005), and check the accuracy of your model again to see if any of the dropped values impact your predictability.

In this case, the score remains the same so it's safe to say you can remove any variables below that threshold:

![Feature Importance after Dropping Values](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/feature_importance2.png)

### Removing Redundant Features:

Another great insight is to use a correlation matrix along with a dendrogram to identify any redundant features:

![Feature Importance after Dropping Values](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/dendrogram.png)

We can see that there are a lot of variables that are very closely related (e.g. saleYear & saleElapsed, Grouser_Tracks & Coupler_System, etc.).  We can use a similar method that we used in feature importance and test to see the models accuracy when these features are removed.

There doesn't appear to be any major differences so it is safe to remove these variables.

**We are now at a point where we have reduced the models dimensionality without sacrificing any accuracy.**
