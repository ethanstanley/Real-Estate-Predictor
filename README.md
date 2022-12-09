## Introduction

The housing market is an intimidating and oftentimes, a hopeless market for students, young workers, and families. It is a story of high demand, low supply, and high prices. Especially in urban hubs, where thousands of people are constantly in search of places to live or rent. As we approach graduation, and will soon look to renting an apartment or house, we have been thinking about how to analyze whether we are paying the right price for living accommodation. A real estate website, such as Trulia or Zillow will give you hundreds of results with just a click. Expensive prices are thrown into your face, with almost no information besides beds, baths, square feet, and the address shown. Put quite simply, it’s overwhelming.

After taking a class on working with Big Data and Data Science, we turned to code. We thought to ourselves if we could make a model that is able to learn how a real estate website such as Trulia prices their houses, we would be able to give the model information about a listing and get a predicted price. We could compare this predicted price with the listed price, and make smart, informed economical decisions on whether a listing is overvalued or undervalued on the real estate site. We also knew that if we approached this problem, we would be able to create meaningful visualizations that would allow us to learn more about the housing market in a specific city and learn more about the importance of features for housing accommodation to the price of it. 

In order to turn our big ideas into a reality, we turned to the most populated city in the United States, New York City (NYC). We thought that if we could get valuable insights from the most populated city in the United States, we could apply our model elsewhere. Additionally, we speculated that NYC would likely have the most accessible data and the largest amount of training data for us to take advantage of. In order to get the data, we turned to Trulia. Trulia is a real estate website for buying and renting properties and contains listings for the majority of the houses in the market. When assessing real estate websites, we also looked into the practicality of scraping data from there, and Trulia looked to be a practical choice. The component-based structure implemented on the website, where each listing contained similarities, seemed to be a great option for taking in a large quantity of data. After deciding on our city of choice and our real estate site to analyze, we were excited to begin analyzing the housing market. But first, we needed to get our data.

## Data Scraping

There is a plethora of data already existing on the internet through publicly available platforms such as Kaggle. If we looked, we could likely find a data set that consists of relevant and practical housing information for a city, such as NYC. However, we plan on using our code and algorithm to make real-time housing decisions, and for such, we need to be able to scrape data from the Trulia website itself. To complete this, we turned to Beautiful Soup, a python package for parsing HTML and XML documents. We began by looking at a specific house listing, and examining the HTML code. 

For a specific house listing, we noticed a few key factors. First, every house listed contained data on the number of bedrooms, number of bathrooms, square footage, the address, and the price. This is because every house was in the format shown below:

<img width="339" alt="Screen Shot 2022-12-09 at 3 36 42 PM" src="https://user-images.githubusercontent.com/106160715/206727903-458a8c7b-47ff-4b41-abd6-aa01069cfc2d.png">


These became the core features we first scraped, since they appeared for every house. After parsing through and investigating the HTML, we found that the bathrooms, bedrooms, and square footage were all in ```<div>``` blocks, where the label was ```data-testid=”home-summary-size-[type of feature]```. We used Beautiful Soup to extract the html in that specific div block, and then created a regular expression to capture the number of whichever intended feature we were searching for from the html block. 

The code for this looked as following:

<img width="728" alt="Screen Shot 2022-12-09 at 3 37 16 PM" src="https://user-images.githubusercontent.com/106160715/206727980-0d9a9985-0b2e-4fe4-873c-01a347b43a8d.png">

We were able to perform this to get the number of bedrooms, number of bathrooms, and square footage. We performed similar operations to get the price of the house, by finding the html block it was located in, and parsing it out through Beautiful Soup and regular expressions. We extracted the zip code from the specific url for the listing, since every listing url had the zip code embedded in the same location. The line of code to receive that was: ```url.split("-")[-3]```. Finally, we identified that for every listing, there was a feature block that showed unique features about the listing. This was shown as:

<img width="523" alt="Screen Shot 2022-12-09 at 3 38 02 PM" src="https://user-images.githubusercontent.com/106160715/206728037-8118c0df-68aa-42ae-bd76-195d02277a86.png">

However, for every listing, different features were shown, so we created a feature dictionary that contained all of the features for every listing. This was performed by finding that these features were embedded with a ```<span>``` tag and a ```class="Feature__FeatureListItem-sc-w1mxt5-0 gmLKqq"```. By iterating over every result in a find all command, we were able to simply extract the features from the section shown above. With that, we scraped the information for one listing. Next, we had to figure out how to perform this over every house. 

We first noticed that there were multiple pages of listings that showed up with a simple search, such as “New York, NY.” To our aid, each page came with a url of the form “https://www.trulia.com/NY/New_York/num_p/" where the page number was inserted into the url where ```num``` is the page number. We iterated over the first 35 pages, and for each page, we used Beautiful Soup to get the link for every listing shown. This was done similarly to the code above, through finding the embedded tag and label, and then using a find all command to get all results. We then passed every link through the parsing method outlined above to get the specific features for every listing. 

However, most websites have safety protocols implemented, to prevent DDoS attacks and malicious hacking. When testing our scraper on smaller quantities of data, we ran into issues with captcha errors and too many request errors. To combat this, we utilized a VPN to change the location of our search pings. This worked in the short term, to allow us to ensure that our scraper was getting the proper data. In order to get a large data set however, we needed to use an unique approach to scrape data over a long period of time but not get flagged or blocked by the website. To combat this, we implemented a sleep period between each search period. In addition, to prevent repetition and detection of repetitive patterns, we decided to make it sleep for a random number between 5-10 seconds every iteration. With this approach, we were able to scrape in data for 1133 listings. We decided that was a large enough sample size to analyze and work with. Next, we processed this data. 

## Data Processing

The first part of processing the raw data consisted of adding a borough feature to the dataset. Location is a key feature when predicting the price of a property. There were a total of 180 Zip Codes across the 1133 properties. These zip codes provide differences in price on a relatively small spatial scale. In order to further capture differences in price on a larger spatial scale, each zip code was mapped to one of the five New York City Boroughs: the Bronx, Brooklyn, Manhattan, Queens and Staten Island. The boroughs were encoded by creating zip code ranges for each borough and assigning each zip code in the dataset a borough based on these ranges. Initially, the livability score from the AARP livability index webpage was also going to be added as a feature for each distinct zip code. However, it was realized that the livability score was based on borough, not specific zip code. Resultantly, livability scores were left out in place of the added borough feature. 

The second part of processing the data was imputing missing values with the features mean or median values. Unfortunately, the Trulia scraping did not turn out a perfect dataset and ,when predicting if a house is undervalued, not all models can handle missing values. As a result, the missing values were chosen to be imputed to create a dataset that could be used across all model types. The data imputation was accomplished using the pandas.fillna method on all columns in the dataset containing missing values. Both the number of stories and year built were imputed using the median value. Lot area, number of beds, and number of baths were imputed using the mean value.

The third part of processing the data was removing any property price outlier values from the dataset. Given the lack of features in the modeling process, outlier values would likely give a misinformed relationship between the property features and property price. Also as undergraduates we are not likely buying 14 million dollar houses (yet). Removing the outlier values was accomplished by creating a box plot of the property prices and identifying an outlier cutoff price values within the plot. Based on the boxplot below, it was decided to remove any property priced over 2.5 million dollars from the dataset. While this removed 106 values from the data, the models should now be able to better map property features to price. 

<img width="316" alt="Screen Shot 2022-12-09 at 3 38 20 PM" src="https://user-images.githubusercontent.com/106160715/206728252-26fcc07f-7382-46cd-80d0-ccbd7a58b666.png">

The final step of data processing was encoding categorical features as numerical features. Any model that is trained to predict undervalued properties can not ingest categorical data. As a result, categorical data must be converted to numerical data. Converting categories to integers allows for models to create artificial relationships between the categories. To avoid this issue, a common categorical data encoding technique called one hot encoding was used. One hot encoding was implemented using the pd.get_dummies method on the borough and property type features. One hot encoding the borough column was done using the following:

<img width="710" alt="Screen Shot 2022-12-09 at 3 38 44 PM" src="https://user-images.githubusercontent.com/106160715/206728271-fb99d79c-c729-4417-acc2-213cd1b58fc3.png">

One hot encoding creates a new column for every category. For the zip code variable, this would have created 180 new columns in the data set. Resultantly, sklearn’s binary encoding process was used to encode the zip code variable into a total of 12 new columns. Binary encoding the zip code column was done using the following:

<img width="724" alt="Screen Shot 2022-12-09 at 3 39 06 PM" src="https://user-images.githubusercontent.com/106160715/206728298-876120c9-3687-438a-8013-a7ec8ab45d28.png">

## Creating Visualizations 

After we processed all of our data, we were left with a cleaned CSV file that was simple to read, however, very difficult to draw any conclusions from. To start understanding what type of data we were working with and how to continue further analysis on it, we turned to the creation of visualizations to describe our data. We first wanted to analyze which features correlate the most with each other. We focused on features that were present in most listings, so that included bedrooms, bathrooms, stories, year built, days on market, lot area, and price. Since our data was in a dataframe, we simply ran the ```data.corr()``` function to get the correlation between variables. To visualize it well, we used a heat map on the correlation data. The correlation heat map is shown below:

<img width="569" alt="Screen Shot 2022-12-09 at 3 39 25 PM" src="https://user-images.githubusercontent.com/106160715/206728331-dc2b67ee-04d2-452a-99de-77e636caec21.png">

As seen, there is a very high correlation between baths and beds. We were able to also see the many other feature correlations, however, ultimately this is not what we wanted. Since we are designing code to predict house prices, we wanted to look at which variables specifically correlate with price. To do this, we took the price subsection, sorted the values, and created a new visualization.

<img width="321" alt="Screen Shot 2022-12-09 at 3 39 43 PM" src="https://user-images.githubusercontent.com/106160715/206728356-a73d1237-7154-4298-9c37-71323d05f818.png">

Some interesting takeaways we had from this were that baths and beds have the highest correlation of all features with the price. Although these correlations were low, we believe this is due to the immense amount of factors that go into the pricing of one house, and additionally confirmed to us that the use of a more complex network to analyze this data would be necessary. An interesting observation was that Lot Area negatively correlated with price. We believe this is due to apartments in Manhattan, where prices are the highest but square footage the lowest, being compared with houses in Staten Island where square footage is likely higher, but house price will be cheaper than in Manhattan. To further investigate the house price by borough, we first began by breaking down our dataset by borough.

The next helpful visualization created was a simple breakdown of the listings by borough. 

<img width="371" alt="Screen Shot 2022-12-09 at 3 40 04 PM" src="https://user-images.githubusercontent.com/106160715/206728386-2f275904-5db5-4217-ab00-7de96afc60e8.png">

This breakdown led us to understand that the majority of our data was coming from three boroughs in particular, with Queens, Brooklyn, and Manhattan containing the majority of the listings. 

Next, we compared the housing price distributions between each of these boroughs:

<img width="746" alt="Screen Shot 2022-12-09 at 3 40 29 PM" src="https://user-images.githubusercontent.com/106160715/206728399-91030054-affa-4e78-9074-92eb65483060.png">

These histograms are normalized by density so the amount (y axis) is relative to the other graphs. They also share an x axis, so the price is distributed between $0 and $5 million. Any houses above that price were removed since there were only around 20/1133 that fit that categorization. After looking at these histograms side by side, it is clear that Manhattan and Brooklyn are the two most expensive boroughs by far, with an almost even distribution that spreads over the whole x-axis. The distribution for the Bronx is skewed left significantly, indicating that most people that live in the Bronx have living accommodations that are cheaper than any other borough. The distributions for both Queens and Staten Island show a dense section around their respective mean prices. After these visualizations, we felt ready to create our models.

## Models

Three models were trained to predict price from the given features: a linear regression model, a random forest model, and a vanilla neural network model. Before training the models, the data was split into training and testing data. In order to maintain a relationship between the predicted values and their original indices in the dataframe, a train/test split function was created. The function iterated through the data and took four properties as training data and then one property as testing data, ultimately creating a controlled shuffling process. This method of splitting the training and testing data yielded 821 training data samples and 206 testing data samples. The code for this function looks like this:

<img width="548" alt="Screen Shot 2022-12-09 at 3 40 55 PM" src="https://user-images.githubusercontent.com/106160715/206728433-117b983b-0ad2-4ed0-b793-d16c9b682739.png">

Sklearn Linear Regression Model:
The model yielded a training absolute error of 253,719 and a testing absolute error of 275,710. The plot below shows the predicted house prices vs. the true house prices. There was a relatively linear relationship between the true and predicted values, demonstrating that the model was picking up on features that lead to higher house prices within the data. 

<img width="421" alt="Screen Shot 2022-12-09 at 3 42 17 PM" src="https://user-images.githubusercontent.com/106160715/206728542-f2bd87be-282d-4392-aa8d-a3edcdde1aa5.png">

TensorFlowVanilla Neural Network Model:
A neural network with the following architecture was created:

<img width="427" alt="Screen Shot 2022-12-09 at 3 42 35 PM" src="https://user-images.githubusercontent.com/106160715/206728576-e1259cad-9e71-42e2-9a25-cd051006c385.png">

The model yielded a training mean absolute error of 261,777 and a validation mean absolute error of 258,269. The training and validation mean absolute error progression across epochs is shown below:

 <img width="394" alt="Screen Shot 2022-12-09 at 3 43 04 PM" src="https://user-images.githubusercontent.com/106160715/206728602-182c0295-9eb2-446f-b5ae-b00d3e1ed4f8.png">

When evaluated on the testing data, the model yielded a mean absolute error of 278,305. 

Sklearn Random Forest Regressor Model:
The model yielded a training absolute error of 58,485 and a testing absolute error of 145,126. The plot below shows the predicted house prices vs. the true house prices. There was a  linear relationship between the true and predicted values, demonstrating that the model was picking up on features that lead to higher house prices within the data, even more so than the linear regression. 

<img width="399" alt="Screen Shot 2022-12-09 at 3 43 24 PM" src="https://user-images.githubusercontent.com/106160715/206728627-9eebd530-4444-40de-b1a7-04a069f37aa1.png">
From the random forest regressor, the variable importance for each feature was extracted. It can be seen from the below plot that the number of baths played a significant role in determining the predicted house price.

<img width="712" alt="Screen Shot 2022-12-09 at 3 43 54 PM" src="https://user-images.githubusercontent.com/106160715/206728791-fa127432-19c2-48dc-b279-b8078dbf868b.png">

## Conclusions

In conclusion, the random forest model yielded the best performance across the three models. In general, the ensemble method utilized in the Random Forest algorithm limited any overfitting without any direct tweaking of the model. In contrast, the neural network requires a lot more tweaking to optimally perform and in general requires a lot more data than was scraped to properly train. Compared to the linear regression model, the random forest model will have many more parameters leading to a deeper understanding of the relationship between the provided features and property price. 

Overall, we were surprised by the results of the random forest regression model. The limited number of properties and features did not lend themselves to accurate price prediction. However, the model performed within a respectable margin of error based on the 2.5 million dollar property price range. In order to increase the performance of the model, more properties and more features would likely need to be scraped. Another means of increasing model performance is to train different models across different boroughs. From the presented visualizations, it is clear that each borough is distinct from one another. Training a model for each borough would factor in these distinct differences to better account for locational discrepancies in price. The only downside to this approach is that the models are now not as generalizable to other cities.  

There is certainly a lot of room for improvement on this project and we are excited to hopefully continue to improve upon these models. First, we ultimately ended up replacing NaN cells with the median or mean of the category, which would negatively impact any edge cases. For a future step, we plan on implementing a model that functions with NaN cells embedded, and that will allow us to also incorporate more features that we did not ultimately use because of lack of data. Additionally, we plan on exploring and changing the structure of the Neural Network model. We believe that changing the model structure will positively affect performance. That performance will also naturally improve as we train on more data, so that is a next step for us as well. 

Next, we plan on looking at the overvalued and undervalued houses that our models currently identify, and analyzing them to understand why they were predicted to be higher or lower than they were priced. As the undervalued houses are our target identification group, this process will be vital in identifying the success of our models. Finally, we want to build on our earlier observation about price differences in boroughs and try to create models custom to each borough to test if that model performs better. All in all, we were very satisfied with the results of this project. We feel confident in our abilities to scrape data, process data, analyze data, and generate models to predict results after this project. 

Thank you for reading. 


