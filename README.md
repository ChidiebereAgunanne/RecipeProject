# Exploring the Correlation Between Prep Time and Recipe Rating

Author: Chidiebere Agunanne

## Overview

This data science project, conducted at UCSD focuses on exploring the relationship between the average rating of a recipe and the amount of time it takes to prepare the recipe in minutes

## Introduction

We often turn to ratings to judge whether something is worth our time—be it a movie, a product, or a recipe. On online cooking platforms, average user ratings offer a quick way to assess a recipe’s quality and appeal. But what if the time it takes to prepare a recipe influences how it's rated? If there's a correlation between prep time and average rating, it could point to a hidden bias—suggesting that ratings may not truly reflect a recipe’s actual value or taste. This project investigates that correlation to determine whether longer cooking times lead to higher satisfaction, or if faster recipes are more favorably received. Such research is important because if ratings are influenced more by convenience than by the actual quality of the recipe, then they may mislead users seeking genuinely good meals. This potential bias can affect how recipes are ranked, discovered, and chosen—ultimately shaping what people cook and eat. By uncovering this relationship, we can better interpret user ratings, encourage more informed recipe selection, and highlight the need for more nuanced evaluation systems on cooking platforms.

The first dataset, `recipe`, contains 83782 rows, indicating 83782 unique recipes, with 10 columns recording the following information:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |

The second dataset, `interactions`, contains 731927 rows and each row contains a review from the user on a specific recipe. The columns it includes are:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

**Given the datasets, we are investigating whether the the preperation time of a recipe impacts the average rating that recipe is given.** To start this inverstigation we will start by cleanifn the data set and conduct explorartory data analysis in order to gaina  basic understanding of the information the dataset hole. I will then analyze the missingness mechanisms and dependency of the data sets. Finally i will build a model that predicts the rating of a recipe. 

## Data Cleaning and Exploratory Data Analysis

To make our analysis of the dataset more efficient and convenient, we conducted the following data cleaning steps.

1. Left merge the recipes and interactions datasets on id and recipe_id.

   - This step helps match the unique recipes with their rating and review.

1. Check data types of all the columns.

   - This step helps us evaluate what data cleaning steps are appropriate for the dataset and if we need to conduct data type conversion.
   - | Column             | Description |
     | :----------------- | :---------- |
     | `'name'`           | object      |
     | `'id'`             | int64       |
     | `'minutes'`        | int64       |
     | `'contributor_id'` | int64       |
     | `'submitted'`      | object      |
     | `'tags'`           | object      |
     | `'nutrition'`      | object      |
     | `'n_steps'`        | int64       |
     | `'steps'`          | object      |
     | `'description'`    | object      |
     | `'ingredients'`    | object      |
     | `'n_ingredients'`  | int64       |
     | `'user_id'`        | float64     |
     | `'recipe_id'`      | float64     |
     | `'date'`           | object      |
     | `'rating'`         | float64     |
     | `'review'`         | object      |

1. Fill all ratings of 0 with np.nan.

   - Rating is generally on a scale from 1 to 5, 1 meaning the lowest rating while 5 means the highest rating. With that being said, a rating of 0 indicates missing values in rating. Thus, to avoid bias in the ratings, we filled the value 0 with np.nan.

1. Add column `'rating_per_recipe'` containing average rating per recipe.

   - Since a recipe can have numerous ratings from different users, we take an average of all the ratings to get a more comprehensive understanding of the rating of a given recipe.

1.  Drop id column containing the recipe id

    - Since this is represented by the `'recipe_id'` as well i deleted a column so no information will be repeated

1. Change the object type of `'recipe_id'` to a string

    - In the dataset `'recipe_id'` is being used as a nominal data type to represent a recipe so I changed it to reflect that

1. Add column `'duration_class'` 

    - `'duration_class'` was calculated  categorizing each recipe's preparation time (minutes) into one of three classes—Short, Medium, or Long—based on the 25th and 75th percentiles (quantiles) of the data. Recipes at or below the 25th percentile were labeled "Short," those between the 25th and 75th percentiles were labeled "Medium," and anything above the median was labeled "Long."

1. Add column `'log_minutes'`

    - `'log_minutes'` was added because the minutes column contained a wide range of values with significant outliers—some recipes had extremely long preparation times that skewed the distribution. To address this and make the data more suitable for analysis and modeling, I applied a logarithmic transformation and created a new column called 'log minutes'. This transformation reduces the impact of extreme values, compresses the scale, and helps normalize the distribution, making patterns in the data easier to observe and interpret.

1. Add column `'num_ratings'`

    -  `'num_ratings'` was added because in the next step of data cleaning i delete duplicate rows from the df, doing this will delete values from the `interactions` dataframe in order to preserve some of this data the column contains the number of times each recipe has been rated (excluding missing ratings). Recipes with no ratings are assigned a count of 0.

1. Drop duplicate `'recipe_id'` rows:
    
    - Duplicate rows can distort the analysis by artificially inflating the frequency or weight of certain data points. For example, if a recipe appears multiple times in the dataset with the same information, it may: Skew summary statistics (like averages or counts), Create misleading trends or correlations, Bias models that assume each row is an independent observation. By removing duplicate rows, I am ensuring the dataset accurately reflects the true distribution of values, leading to more reliable insights and valid conclusions during exploration and modeling

#### Result
Here are all the columns of the cleaned df.

| Column            | Data Type   |
|-------------------|-------------|
| `'name'`              | object      |
| `'minutes'`           | int64       |
| `'contributor_id '`   | int64       |
| `'submitted'`         | object      |
| `'tags'`              | object      |
| `'nutrition'`         | object      |
| `'n_steps'`           | int64       |
| `'steps'`             | object      |
| `'description'`       | object      |
| `'ingredients'`       | object      |
| `'n_ingredients'`     | int64       |
| `'user_id'`           | float64     |
| `'recipe_id   '`      | object      |
| `'date'`              | object      |
| `'rating'`            | float64     |
| `'review '`           | object      |
| `'rating_per_recipe'` | float64     |
| `'duration_class'`    | object      |
| `'log_minutes'`       | float64     |
| `'num_ratings'`       | float64     |


Our cleaned dataframe ended up with 234429 rows and 20 columns. Here are the first 5 rows of our cleaned dataframe for illustration. Since there is a lot of columns for the merged dataframe, we selected the columns that are most relevant to our questions for display. Scroll right to view more columns.

| name                                 |   recipe_id |   minutes |   log_minutes |   rating |   rating_per_recipe | duration_class   |   num_ratings |
|:-------------------------------------|------------:|----------:|--------------:|---------:|--------------------:|:-----------------|--------------:|
| 1 brownies in the world    best ever |      333281 |        40 |       1.60206 |        4 |                   4 | Medium           |             1 |
| 1 in canada chocolate chip cookies   |      453467 |        45 |       1.65321 |        5 |                   5 | Medium           |             1 |
| 412 broccoli casserole               |      306168 |        40 |       1.60206 |        5 |                   5 | Medium           |             4 |
| millionaire pound cake               |      286009 |       120 |       2.07918 |        5 |                   5 | Long             |             1 |
| 2000 meatloaf                        |      475785 |        90 |       1.95424 |        5 |                   5 | Long             |             2 |

### Univariate Analysis

For this analysis we examined the distribution of minutes in the dataframe. That is we examined how long most recipes in the dataset are. The plot shows that most recipes cluster between 1.0 and 2.0 log10 values which corrspons to about 10 to 100 minutes. The peak of the histogram is just below 2, meaning a large number of recipes take between ~30 and 100 minutes. There are very few recipes with log-transformed prep times above 3 , indicating the presence of outliers, but they are rare

<iframe
  src="assets/minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

In this analysis, we examined the distribution of average ratings per recipe in the DataFrame. Since the `'rating_per_recipe`' column contains floating-point values (as it's an average), I rounded the ratings to create clearer groupings for the plot.The graph reveals that most recipes have an average rating of 5, followed by a significant number around 4. In contrast, very few recipes have ratings below 3. This indicates a highly skewed distribution toward the higher end—particularly 5—suggesting that users tend to rate recipes very positively, and that low-rated recipes are underrepresented.

<iframe
  src="assets/average_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

The plot shows the relationship between the `'minutes'` column (scaled down on the y-axis using a logarithmic scale) and `'rating_per_recipe'` on the x-axis. The data points are widely scattered across all rating values, indicating that there is no strong correlation between preparation time and average rating. In other words, there is no clear linear or obvious relationship between the two variables. Most of the data points are clustered around ratings 4 and 5, which aligns with the overall distribution in the dataset, where most recipes are highly rated. Additionally, there is a wide variation in preparation times at every rating level. This suggests that a long prep time doesn't guarantee a high rating, and a short prep time doesn't necessarily lead to a low one.

<iframe
  src="assets/time_v_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

In this section, I explored the relationship between cooking time (in minutes) and the number of non-missing ratings (NaN values excluded) received per recipe. To do this, I created a pivot table using the `'minutes'` column as the index and `'num_ratings'` as the values. After grouping the data, I applied aggregation functions including mean, median, min, max, and count to summarize the distribution of ratings across different preparation times.

|   minutes |   ('mean', 'num_ratings') |   ('median', 'num_ratings') |   ('min', 'num_ratings') |   ('max', 'num_ratings') |   ('count', 'num_ratings') |
|----------:|--------------------------:|----------------------------:|-------------------------:|-------------------------:|---------------------------:|
|         0 |                  3        |                         3   |                        3 |                        3 |                          1 |
|         1 |                  3.0655   |                         2   |                        0 |                       38 |                        229 |
|         2 |                  2.89978  |                         2   |                        0 |                       20 |                        908 |
|         3 |                  2.93196  |                         2   |                        0 |                       86 |                        485 |
|         4 |                  3.17603  |                         2   |                        0 |                       36 |                        267|

The graph shows that as cooking time increases, the number of ratings given to a recipe decreases sharply. This suggests that longer recipes are rated less frequently, making them more susceptible to outliers and potentially less accurate average ratings. The mean line reveals that the average number of ratings remains relatively consistent across most preparation times, with occasional dips and spikes in longer-duration ranges—likely due to smaller sample sizes. I chose to display only the mean and count aggregation functions to avoid cluttering the graph. The other metrics (like median, min, and max) tend to follow similar patterns and would have added little new information while making the chart harder to interpret.

<iframe
  src="assets/minutes_v_numRatings.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>