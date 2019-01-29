# Using a Random Forest for Price Prediction and EDA

## The Data
This project analyzed data on bulldozer prices and included 53 predictors on a given bulldozer including the year, sale date, enclosure type (open, enclosed, enclosed w/ AC, etc.)

![Bull Dozer Data](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/data.png)

## Cleaning the Data

The dataset includes a lot of categorical variables in the form of strings, as well as variables that have a lot of missing values.

I don't want to delete any of the data so early in the project, so I used a function that will convert all string values into categories and use the category codes to feed into the model.

The next thing I did with continuous predictors that had missing values is fill the missing values with the columns median, but also create a new column that indicates whether that row had a null value for that column. This way the model can identify if a missing value is significant.

## The Model

For this project I used a **random forest** because of its flexibility to different datatypes and patterns, as well as its ability to provide insight using feature importance, partial dependence plots, and prediction interpretations using ELI5.

## Evaluating Features

Using Feature importance you can see that the year of the bulldozer is very predictive of its price:

![Initial Feature Importance](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/feature_importance.png)

Feature Importance is calculated by randomizing a certain variable and calculating the effect this randomization has on a models prediction score.

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

## Insights

### One Hot Encoding

Not only is it important to look at the feature importance through a categorical lense, but you can also find insights by breaking apart certain columns using one hot encoding.

For this section, any feature that had 7 unique values or less, I used pd.get_dummies to see if any specific feature values are predictive of price:

![Feature Importance after One Hot Encoding](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/one_hot_encoding.png)

### Partial dependence

It's great to know what features are important to a models prediction (i.e. a bulldozers expected price).  However, it is also important to know **how** a feature affects a models prediction.  One way to do this is prediction interpretation (which I'll explain next), but another way is through calculating partial dependence, which shows how a prediction is affected holding all other variables constant.

#### Year Made:

![Year Made Partial Dependence](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/pdp1.png)

From the graph above, we can see that the newer the model, the higher the sale price of a bull dozer.

#### Product Size:

![Product Size Partial Dependence](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/pdp3.png)

From the graph above, we can see that the bigger the bull dozer, the higher its sale price.

#### Enclosure (Enclosure EROPS w AC):

![Enclosure Partial Dependence](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/pdp2.png)

What this graph is telling us is that when a bulldozer is enclosed as opposed to open with an Air Conditioning unit, this is predictive of a higher sale price.

### Prediction Explanation:

Another way to see how our features can affect a bull dozers sale price is using ELI5 to explain what features contributed to a specific prediction (i.e. an individual row):

![Prediction Interpretation](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/large_2007_wAC.png)
#### This is a large, 2007 bull dozer that comes with AC, it's sale price was $105,000

![Prediction Interpretation](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/mini_1998_woAC.png)
#### This is a mini, 1998 bull dozer that has no AC, it's sale price was $

![Prediction Interpretation](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/mini_1988_woAC.png)
#### This is a mini, 1988 bull dozer that has no AC, it's sale price was $11,000

These further confirm what we saw earlier, that size, year, and AC are good predictors of a bulldozers sale price.

![Sale Medians](https://s3.amazonaws.com/chermsbucket/rf_imp_folder/summary.png)
