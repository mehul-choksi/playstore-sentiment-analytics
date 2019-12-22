## Introduction and Story

With the ever increasing growth in demand and production of mobile applications, app stores are constantly looking for effective metrics to select and display the best applications to its users. The current system in place is a numeric based rating system, where the users can rate an application on a scale of 1-5. Subsequently, the average is calculated, which then serves as a metric. The second challenge to the system is the presence of fraudulent applications, particularly common on android operating systems. These applications lure in innocent users to watch advertisements, guarenteeing them certain rewards which are never doled out. In reality, these applications are merely adware. The applications also manipulate the users into giving them a 5 star rating. This creates a high numeric rating for the application. However, if one goes through its text reviews, the negative sentiment is clearly evident. We present a system will be able to detect such anomalies, and be effectively used for weeding out such applications.

However, this method is often suseptible to various malpractices. Two most common exploits on this system are:
	1. Botting: Developers write scripts, to create bots, which are essentially fake accounts, that are able to give 5 star ratings
	2. Tricking the user into giving 5 stars: A lot of applications tend to manipulate users, by offering them certain dubious benefits, if they give a 5 star rating

This often inflates the ratings of applications that are otherwise, mediocre - or even bad.

## Solution

	This project attempts to solve this problem by mining the text reviews, doing sentiment analysis, and then calculating the ratings. It then considers the playstore rating, and is also able to find out the deviation in the ratings.
	At the end of the project, we will have a fully functioning, end to end application that will create a simple dashboard to present the analytics of the application.

	For demonstration purposes, our application will be relying on google play store as a source of information.

## Architecture

	Components of the system:
		1. Web scrapper: Responsible for scrapping the reviews on applications, and storing them in a text dump
		2. Parser: Cleans and extracts the data from the raw text dump, and stores it in a csv file
		3. NLP module: Build, train and save a model on a large set of reviews. This will be used for predicting the sentiments of future reviews.
		4. Web application: A simple web application, that exposes an endpoint for the user to use the application. 
		5. Visualization & stats: Plot simple visualizations, using plotly and wordcloud. Calculate custom ratings.
		
## Tech stack + Why 

	Choice of programming language: Python
	Why: Minimum boiler plate code, built in support for creating web apps, scrapping, parsing and NLP

	Scrapper: Python Selenium
	Why: Quick and convienient to use, faster than traditional scrappers

	Web application: Python flask
	Why: Allows you to create a web application with minimum efforts, great for prototyping

	NLP: Sklearn, SpaCy
	Why: Powerful libraries, widely supported, efficient 

## Prerequistes
	You will require python3+,pip, sklearn, numpy, pandas, spacy, virtual environment. All other dependencies can be installed from requirements.txt or the instructions.

## Scraping

	The goal of this module is to extract the text content from playstore pages, and dump it in a text file.

	Dependencies:
		1. Python selenium:
		2. Geckodriver (or equivalent)

	Selenium is a software that is used to automate browsers. While selenium is commonly used for testing and administration, we will be using it as a scrapper.
	Approach for extracting: Load a web page, select all, copy, paste into a text file.
  
## Extracting

	Now that we have a raw text module, we need to parse it. Now this is how our text dump looks like
	
	 Observations:
		1. The first few lines are irrelevant, they do not contain reviews.
		2. After that there is a recurring sequence of: User name, Date of Review, Review likes, Review
		3. For the reviews with no likes, review likes will not appear. We will have to handle this

	Approach:
      1. Traverse the header of file (first 9-10 lines)
		  2. Check if the line starts with a name of month -> Indicates a date -> Indicates a review
		  3. Note down date,like count,review
		  4. Write in csv

## Text processing: 
		1. Apply python regular expressions to replace all non ascii characters with blankspace.
		2. Remove extra whitespaces
		3. convert all the words into lower case:
		4. Below is the utility function for the same
	
	
## Preparing dataset
	1. To prepare a well rounded dataset, it must have a vast number of reviews, from a vast number of applications
	2. The google playstore provides reviews to its application if you have the application id.
	Therefore, we need to mine a large number of application ids first.
	3.There is a top charts section in google play. This includes: top free, top paid, top grossing, etc.
	4. Each of these have numerous applications listed in them. We will write a scrapper that will visit each of the the top charts pages, and extract application id from the applications listed on that page.
  
  
	1. [Run review scrapper for each id]
		After extracting the application ids, our next task would be running our review scrapper for each of the application ids. It scrolls ~20 times, to accumalate about 500 reviews/application.

	2. Run extractor on raw text file
		Now run the extractor function on each raw file, and convert it into a cleaned csv file.
	3. Merge
		Since the ratings are non textual, they are mined and stored seperately. We need to combine them with the reviews, because these ratings are the labels/target classes of the reviews. Without them, our model cannot differentiate between positive/negative reviews.
		
	Finally, combine all the seperate application reviews csvs into one large csv. This will be the dataset used to training our model.
## NLP
	Our training set consists of 100k+ reviews and ratings. We will now feed these to model. The model with be able to associate positive/negative sentiments with the words. We have used multinomial naive bayes for sentiment analysis. This model assumes that each word is independent, and contributes equally to the outcome. Despite its simplicity, it is still one of the best algorithms for text processing.

## Saving and loading model 

	When we trained our model, it took over 30 mins to do so. It is therefore not feasible to have this kind of computation done each time an application starts. Instead, we use Python's joblib model to save our model as a file. This file can be loaded and used in any python application having joblib dependency.

	
## Web application with flask

	The goal of the web application is to provide the user with an interface, using which he can interact with the application. The webpage of the application will ask the user to enter the link to playstore application. 
	It will then be responsible for mining the reviews live, feeding them into a model, and plotting the responses as a simple dashboard.

	Flow of the web application
	1. Link of application => Feed into the input box
	2. Scrapper invoked with link fed => Raw text dump acquired
	3. Extractor => Cleaned csv file
	4. Reviews => Feed into the sentiment analysis model => Get counts of positive and negative sentiments.
	5. Visualizations => Pie chart representation of sentiments => Positive and Negative word clouds
	6. Stats => Calculate the custom rating, deviation from playstore

# Evaluation
	Test 1: An overrated application, with high ratings:

	![overrated](https://github.com/mehul-choksi/playstore-sentiment-analytics/blob/master/images/illegit_1.png)

	Test 2: An authentic, high quality application:

	![Legit](https://github.com/mehul-choksi/playstore-sentiment-analytics/blob/master/images/legit_1.png)
	
	![WordCloud](https://github.com/mehul-choksi/playstore-sentiment-analytics/blob/master/images/legit_wordcloud.png)
## Future Scope
	1. The application reviews contain emoticons. While these are non ascii characters, our preprocessor simply eliminates them. Emoticons are, in essence are a very vital source of the sentiment. An effective preprocessor should be able to map the emoticon to english emoticon word. This would further improve the efficiency of the sentiment analysis.

	2. The number of review likes, could also be given a weight. Instead of giving each review equal weight, we would apply a transformation like logarithmic transformation on the review likes, and then give them a weight.

	3. Try to develop a model which is able to consider n-grams, instead of single words. Our current model, doesn't work well on statements like 'I don't like the app', mainly because it considers words as seperate, indivivual tokens.

	4. Identify duplicate reviews: No 2 reviews written by humans are exactly the same. After preprocessing the text, if the review matches a previous review exactly, then discard it.
