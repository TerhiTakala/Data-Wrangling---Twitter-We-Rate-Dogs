# Data Wrangling - Twitter data - WeRateDogs

## 1. Introduction
This is a summary of the data wrangling efforts made to gather, assess, and clean the WeRateDogs data as part of the Udacity Data Analytics Nanodegree.

WeRateDogs is a successful twitter channel that asks people to send photos of their dogs, and then humorously rates and comments the selected images. The rating scale is from one to ten, but because the dogs are good dogs, the ratings excess oftentimes the maximum reaching 14/10, and above if the dog is a hero.

## 2. Data Gathering 

In this part I gathered 3 required sets of data in a Jupyter Notebook titled as wrange_act.ipynb. The data was gathered in the following ways:
	a. Dowloading the twitter_archive_enhanced.csv file provided by Udacity and reading it as Pandas dataframe and storing it under variable ‘df_twitter_archive’

	b. Programmatically downloading the file image.predictions.tsv that is hosted on the Udacity’s server, using Python’s request library and reading it as Pandas dataframe and storing it under variable ‘df_image_predictions’.

	c. Querying Twitter’s API for the tweet IDs appearing in the df_twitter_archive for each tweet's JSON data using Tweepy Library. Each tweet’s JSON data was written in its own line, and finally saved as text file titled tweet_json.text. This text file was then read as a Pandas dataframe and stored under variable ‘tweet_selected_attr’, with attributes tweet_id, favorites, retweets and timestamp. The Twitter API keys, secrets, and tokens were hidden from the code in the project submission in order to comply with Twitter’s Terms and Conditions.

## 3. Data Assessing 

In the assessing step of the project I visually and programmatically assessed the data for;
	a. Quality issues, which are also called as dirty data. These are content issues like inaccurate, missing, or duplicated data in a dataset

	b. Tidiness issues, which is also called as messy data. These are structural issues in the dataset that make the analysis harder.

### i. Visual assessment:
		1. In the visual assessment step, I got acquainted with the data and tried to understand what the data is about. 
    I also started to get an idea what columns and rows I would be using in my analysis. I displayd all 3 data frames in their entirety in the data frames they were gathered into.

### ii. Programmatic assessment:
		1. In order to assess the data programmatically and check for data quality issues, I used Python’s methods .info, .value_counts, .duplicated(). Identified quality issues: Twitter Archive
			a. The data includes retweets and not only original tweets with ratings – need to remove the items that have retweeted_status_id is non_null. There are 181 retweets and 78 replies that re not needed in the dataframe
			b. Timestamp is object instead of datetime
			c. Names of the dogs are not complete or have unexpected values such as ‘such’, ‘the’, ‘this’, ‘very’, ‘unacceptable’, etc. these non-dog names are in lowercase and proper dog names are capitalized. Name 'O' is incorrect and the correct name is O'Malley. There are also values 'None' that should be replaced by 'Nan'
			d. Some rating_denominator values are not valid. Should be 10, and if higher it could be a total rating of a group of dogs, or wrongly extracted data
			e. Some rating_numerator values are not correct. Rating numerators should generally not be higher than 14, although according to WeRateDogs there are some cases with ratings like 420/10, 1776/10, and 666/10 WeRateDogs
			f. Datatype of rating_denominator should be float
			g. Datatype of rating_numerator should be float
			h. As we will not use the tweet_id column to mathematical calculations this should be object instead an integer
			i. Change the Dogtionary (doggo, floofer, pupper, puppo) ‘None’ values to NaN
			j. There are unnecessary columns that are not needed for the analysis, such as: expanded_url and source 

#### Image Predictions

	a. 324 predictions of the Tweets' images are wrongly predicted to something else than a dog. tweet_id 892420643555336193 for example has as predictions are orange, bagel and a banana

	b. The tweet_id column is an integer and not object

	c. There are 2075 rows in this dataframe, and 2356 in the df_twitter_archive. This means that 281 tweets are missing images

	d. 66 of the image URLs are duplicated, however, it is possible that some of these are retweets, so we will recheck this in a later stage when the dataframes are merged into a master_df

	e. Not all predictions are in same format, some are capitalized, some not 


#### Additional data from Twitter's API

	a. Timestamp is object instead of datetime

	b. There are 2328 rows in the dataframe, which is 28 less than the original ‘df_twitter_archive’ has. This is be because of the errors returned in querying the data from Twitter’s API

	c. The tweet_id column is an integer and not object 


#### Tidiness Twitter Archive

	a. Doggo, loofer, pupper and puppo are one observation and should be one column instead of 4 different columns.

	b. In order to use the rating denominator and the numerator in the analysis, we need to have the one observation in one column instead of 2.

	c. Delete unnecessary columns that are not needed for the analysis, such as: expanded_url and source


#### Image Predictions

	a. There are 3 predictions, however only 1, the best prediction regarding the confidence and the match of being a dog, should be enough for the purpose of the analysis. The priority order is as p1 is greater than p2, and p2 is greater than p3 All 3 dataframes

	a. Tidy data says that one observation unit should belong to a single table. The three tables represent the same kind observation, so, they belong in a single table


## 4. Data Cleaning 

In the cleaning phase of the project I cleaned and organized each of the issues documented assessment phase resulting into a high quality and tidy master_df pandas DataFrame that is easy to use for the analysis. Before starting the cleaning process, I made copies of the data frames and saved them under archive_clean, image_clean, and tweet_clean variables. I also saved the cvs files of each dataframes locally. The cleaning steps were following the process Define, Code, Test. 

#### Here are the cleaning activities:

	a. Merge the 3 dataframes into one master_df
		i. Merged tweet_clean and archive clean using inner join so that the master_df would only contains the rows that have matching values in both of the original DataFrames. This removed the 28 Tweets that do not have the additional attributes queried from Twitter's API

		ii. Merged master_df and image_clean using inner join so that the master_df only contains the rows that have matching values in both of the DataFrames. This removed the 281 Tweets that were missing images

	b. Removed the retweeted items from the dataset by storing only the lines with retweeted_status_id non_null
	
	c. Removed the replies from the dataset by storing only the lines with in_reply_to_status_id non_null to the master_df dataframe

	d. Changed the timestamp datatype to datetime instead of object and renamed the column timestamp_x to timestamp

	e. Changed the datatypes of: i. tweet_id column to an integer instead of object ii. rating_numerator to float iii. rating_denominator to float b. Replaced the dog names that are not complete or are invalid (ones in lower case) by 'NaN', and also 'None' values by 'NaN', and "O" with correct name O'Malley c. Changed the incorrect rating_denominators and numerators to match the values in the text

	d. Converted the rating_denominator and numerator into 1 column, from fraction to a decimal, by dividing the numerator by the denominator

	e. Kept only 1 column for image prediction and 1 column for the prediction confidence choosing the best available prediction result. This was done by creating a function that saved the first available correct prediction to the list that was then stored to variable ’dog_breed’.

	f. Cleaned the dog breeds column that values have a consistent style. Replaced the _ with space and capitalized the first letter of each word

	g. Deleted all the columns that are not needed for the analysis

	h. Changed the Dogtionary (doggo, floofer, pupper, puppo) ‘None’ values to NaN

	i. Combined the doggo, floofer, pupper and puppo to one column instead of 4 different columns, corrected the values that have 2 dog stages, and finally deleted the doggo, floofer, pupper and puppo columns that are no longer needed. Finally, I made final checks for the data frame that all the data is good and clean, and saved the master_df as csv titled twitter_archive_master.csv
