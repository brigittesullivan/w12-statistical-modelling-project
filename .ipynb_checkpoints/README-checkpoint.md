# Final-Project-Statistical-Modelling-with-Python
By : Brigitte Sullivan <br>
Submitted on: Friday, September 1st 2023<br>
Lighthouse Labs Data Science Program <br>

## Project/Goals
Project description and goals: 

* Perform statistical modelling to determine the relationship between the prevalance of "green spaces" / outdoor spaces and city bike stations in Montréal. Specifically, the relationship between number of city bike slots and the common outdoor space categories within 1km of city bike stations. 
* Chose Montréal as city because of its ample outdoor spaces and "picnic"-culture (mentioned to me often annectodally) leading me to hypothesize that there would be a relationship between 
* There was no explicit mention that free bikes is the required independent variable in compass content and felt that number of slots represented overall supply/demand of bikes better than a point in time number of free bikes.

##### **Null hypothesis:** 
There is no relationship between the number bike slots and the number of outdoor POI's (place of interest)

##### **Alternative hypothesis:** 
There is a relationship between number of bike slots and number of outdoor POI's

## Process

The steps taken to complete this project were: 

### 1. Extract Data from City Bikes and Foursquare (Part 1 & 2) 
* Used api get request to gather the data. 
* Gathered locations for all city bikes stations in Montréal, used these station locations to use as the center of the radius search for the Foursquare API request 
* focused on bringing in as much data from city bikes as possible. I (wrongly) assumed this this would improve the likelihood of building a model that was good at predicting the target variable, however it simply created a challenge (discussed later in the challenges section)
* Decided to focus on first gathering data from Foursquare and if time, return to extract yelp data. 
* Chose to focus Foursquare data on any POI's that had the parent category of "Landmarks & Outdoors" (e.g., anything with a categoryid in the 16000's). 
* Removed additional categories as a data cleaning step (seemed simpler to do using a dataframe filter than in the API call)

[Link - City Bikes Extract(Part 1)](https://github.com/brigittesullivan/w12-statistical-modelling-project/blob/main/notebooks/1_city_bikes.ipynb)
[Link - Foursquare Extract (Part 2)](https://github.com/brigittesullivan/w12-statistical-modelling-project/blob/main/notebooks/2_yelp_foursquare_EDA.ipynb)


### 2. Join Data Sets & and initial exploration (Part 3) 

* Joined City Bikes data and Foursquare data based on station location. 
    * City Bikes data had one location per station. 
    * Foursquare data had many POI's for one location since it's possible to have multiple outdoor spaces within 1km of a bike station in Montreal).
* performed QA/data validation steps to ensure the join of City Bikes data and Foursquare data performed as expected. 
    * QA done by taking a sample station and examining what the output should be then confirming that the joined data set contained this output.
* EDA showed no obvious relationship between any of the possible target variables and possible independent variables. 

[Link - Joining Data (Part 3)](https://github.com/brigittesullivan/w12-statistical-modelling-project/blob/main/notebooks/3_joining_data.ipynb)

### 3. Statistical Modelling

**Original approach:** I first attempted simple linear regression on all possible individual combinations between the independent variables and the target variables.  

independent variables:
* outdoor_space_num
* num_parks

with target (dependent) variables:
* free_bikes
* empty_slots
* slots
* ebikes
*At this point, having multiple target variable options was a problem I didn't know I had.*

The highest adjusted r-square value was 0.0055 of all the 8 models I created. All these models were removed from the statistical modelling notebook for clarity. 
Given I had the time, I decided to revise my approach in the hopes that I could find a 'better' model.

**Revised approach:** Decided to restructure the data to have the number of outdoor space by type as seperate columns so that there are more options for independent variables, and narrowed the independent variable to number of slots. 

The steps taken in the revised approach were: 
* Restructure Data
    * pivot the category name column so that each outdoor space category had a column with a numeric value
* Address NaN values
    * pivoting the category name column created many NaN values that needed to be resolved (by filling the NaN values with 0). 
* Perform multivariate linear regression with backward selection using number of slots as dependent variable and each outdoor space category as independent variables. 
    * Target Variable: 
        * number of slots at one station
    * Independent Variables: 
        * Park
        * Playground
        * Monument
        * Farm
        * Garden
        * Dog Park
        * Campground
        * Hiking Trail
        * Landmarks and Outdoors
        * Historic and Protected Site

[Link - Statistical Model (Part 4)](https://github.com/brigittesullivan/w12-statistical-modelling-project/blob/main/notebooks/4_model_building.ipynb)

## Results
Here are the results of the statistical modelling performed: 

* The adjusted R-quare value in the final model(output B) after performing backward selection was **0.192**. 
    * Meaning that 19.2% of the variance in the dependent variable can be explained by the independent variables. 
    * This is considered a fairly weak relationship, and the model's ability to predict the number of slots is not is poor. 
* In model output A (the model previous to the final model "B"), the independent variable Park is technically not statistiscally significant with a pvalue of 0.062 (but very close to 0.05 threshold). When Garden is removed from Model B, the pvalue for Park drops and Park becomes a statistically significant.
* **Historic and Protected sites** is the outdoor space category that makes the largest contribution to the model with the highest coefficient value, followed by **monuments**. Both categories have a P-value of 0.000 indicating high significance.
* Conclusion: 
    * There is a statistically significant relationship between the number of bike slots and the number of these outdoor spaces within 1km of the station (in order of descending significance): 
        * Historic and Protected Sites
        * Monuments
        * Farms
        * Parks
    * although my original thought that parks would be the strongest indicator the model found that there are other outdoor spaces that better predict the number of slots than Parks (like Historic Sites, Monuments, Farms)
     * In model output B, the very small **f-statistic value of 6.18e-35** indicates we can still reject the null hypothesis that there is no relationship between number of bike slots and number of outdoor spaces.
     * In other (slightly less accurate, but more intuitive words): ***there is a relationship between the number of bike slots and the number of outdoor POI's in Montréal***
See images Model Output A and Model Output B in images folder. 

![model output b](https://github.com/brigittesullivan/w12-statistical-modelling-project/blob/643beb8d3caf8ecdb4a7b06a1176b9e15479a99b/images/Model%20Output%20B%20(final%20model).png)</br>

![Model_output_A.png](https://github.com/brigittesullivan/w12-statistical-modelling-project/blob/643beb8d3caf8ecdb4a7b06a1176b9e15479a99b/images/Model%20Output%20A.png)

## Challenges 

The challenges I encountered in this project were: 

* determining the dependent and independent variables too late. 
    * I admitedly wanted to pick "the right variables" that would return the "best" model without any information meant I delayed an important decision for perhaps too long. Not having the question fully formed prior to data extraction meant having too many options for dependent variables and to few options for independent variables. 
* not finding any "good" models during first iteration of modelling and needing to restructure data. 
* The first iteration, the two posible independent variables outdoor space num and num parks violate the multicolinearity assumption since the num_parks variable was a direct subset of the outdoor space num. This was another reason for why I needed to revise the my approach. 

## Future Goals
(what would you do if you had more time?)
* Explore the Yelp API data to see if the results are any different
* Analyse Landmarks & Outdoors as an entire category and/or analyse all parent/meta categories 
* Perform multivariate linear regression analysis on all outdoor categories instead of only the top 10
* change the dependent variable to number of free bikes to see if the model outputs improves