
## <center>Machine Trading</center>

#### <right>(By Dali Souayah)</right>

This report will follow the traditional data science process, which the following:

1. Define the problem
2. Collect the data
3. Clean and process the data
4. select the model(s)
5. evaluate the model(s)
6. Answer the problem

# I. Define the problem

##  1. The real life problem

Financial analysts issue their predictions about stock prices based on financial information.
It is well documented that no active trader can sustainably beat the market. Numerous research has shown that active management is hardly justified. On the long-run, even if if an "alpha" is generated, it is unlikely that it would still survive the deduction of management fees and extra taxes due to active trading. That was the reason for the creation of passive investment through index funds and Exchange traded Funds(ETFs).

One of the reasons active managers find a hard time accurately predict stock prices is emotions. Emotions play a big role in stock trading. A usual situation, whereby a stock looses 5% in a day and then reverts to its original value within the week, is a common occurrence. If the decline was based on some fundamentals, it wouldn't have reverted. That is the emotion playing a role, by over-reacting to some events or announcements. Also there is the element of uncertainty which plays a big role in the volatility of the stock market. Analysts try their best to gauge this uncertainty by asking questions to the executives during press releases, in an attempt to demystify their 8-k reports.

It is based on these perceived market inefficiencies that came the idea of this project. It is clear that the community of financial analysts, stock traders, and portfolio managers is somehow missing on a big part of the available information. Certainly, they are deploying the most sophisticated means to issue their predictions, often based on proven valuation techniques. My assumption is that it is likely that financial analysis has reached its limit in forecasting the movement in stock prices.

The use of machine learning in predicting stock market movement is a nascent area and could push the limits of traditional modeling techniques. Natural Language Processing (NLP) can play a big role in analyzing another part of the companies' disclosures, namely the text or the press releases accompanying the financial/numerical information. Not less important is the evolution of that text and announcements. Traditional techniques (basically reading through and comparing reports) inhibit the discovery of trends that could, otherwise, be telling about a stock going one way or another. Of course, this does not preclude traditional financial analysis. On the contrary, the latter is the basis of the analysis, textual analysis will come as a refinement of it or in some extreme cases, will flag that something is not adding up.

In my opinion, although the information contained in the financial reports is static, its interpretation varies. Interpretation can differ due to many things, including experience, depth of research, human error, capacity limit to learn from/analyze/incorporate past reports/experiences, or just personal bias.  If we consider financial information as a set of numerical and non-numerical information, I would argue that the interpretation of non-numerical information is the one that is most prone to bias and misinterpretation. Unlike numerical information, non-numerical information, i.e. the text that surrounds the numerical information, has much less rules to abide by. Given this freedom of choice, bias prevails, opinions diverge, and volatility increases.

So the data science question here is: What if we can reduce this bias by reducing human interpretation of non-numerical financial information.

## 2. The data science problem:

### Main feature

I propose to analyze the textual information contained in the financial reports through NLP. The analysis will focus on the use of qualifiers that resonate, **negativity**, **positivity** or **uncertainty**.

Toward that end, I propose to train a model, that will learn the past mistakes in interpreting the non-numerical information contained in financial reports (my text feature).

### Target variable

Mistakes of interpretation are defined as the difference between **(1)** the movement in the stock market within **3 days** or **7 days** of the new release and **(2)** the price at which the stock is trading by the end of the period, at which time the previous release is assumed obsolete, as it had been already factored in the stock price. That difference is my target/independent variable, as further noted below:

- `delta_3_np`/`delta_7_np` (binary): the change in price between 3/7 days after the first release and 1 day prior to the next release
- `delta_3_n0/_7_` (binary): the change in price between 3/7 days after the first release and the day of the next release

The idea is to appreciate how that initial information duly or unduly influenced the stock movement. At first, the determination of when this information becomes obsolete is not clear cut, thus I analyzed the correlation with 3 possible dates: **the price the day before the next release**, **the day of the next release**, and  **the day after the next release**. The idea, is of course to be able to forecast those trends with unseen data.

By forecasting a trend (up or down) rather than a specific price, our problem is defined as a classification problem, a binary one.

### data dictionary of secondary features

As will be detailed in the section below, I have collected data about many variables for each company (represented by a ticker), including:
- `text` (str): the text of the 8-k filing: which contains information about important events. This text has been split into 3 categories: negative, positive, and uncertain connotations
- `date` (date_type): date of the 8-k release
- `delta_0_p` (binary): the change in price  between the day of the release and the day prior to the release
- `delta_1_p` (binary): the change in price between one day after the release and the day before the release
- `delta_1_0` (binary): the change in price between one day after the release and the day of the release
- `period` (int): the duration between two consecutive releases
- `an_buy/sell/hold` (int): the number of analysts issuing a buy/sell/hold recommendation
- `an_rating` (float): on a scale from 0 to 5, denotes the attractiveness of the stock
- `an_d_p` (binary): the difference between the analyst's target price and the price prior to the next release
- `an_d_0` (binary): the difference between the analyst's target price and the price at the next release

# II. Collect the data

I collected data from 4 sources:
- **SEC** website using an open source scraper: [SEC-Edagar]('https://github.com/coyo8/sec-edgar'). The SEC data served to collect the filing texts. the webscraper limits the number of filings to be extracted at the last 100.
- **Yahoo finance** website using an open source scraper: [Yahoo_fin]('http://theautomatic.net/yahoo_fin-documentation/'). Yahoo Finance was used to get all data related to prices
- **Bloomberg**. The Bloomberg data was exported into csv formats and used to collect all information related to analysts' stock forecasts, recommendations, and ratings
- **Dictionary of words**, a dictionary of financial sentiment words, from [EDGAR-reports-Text-Analysis]('https://github.com/rohitharitash/EDGAR-reports-Text-Analysis?files=1')

# III. Clean and Process the Data

## 1. Clean the data

The text data comes in a raw format, which needed cleaning and processing. First, I extracted key information using Regex. Key information included, the name of the company, it's ticker, and the date of the filing.
Then, I extracted the body of the filing and mapped it with the dictionary of words.
These matching words were added as the main features to the pandas data frame.
The company names and tickers were changed, whenever needed to match the casing and spelling of Yahoo Finance labeling.

## 3. Feature creation

from the main filing date, several dates were created, namely, the day before and the day after, 3 days, and 7 days after. All these dates have been mapped with the yahoo data to provide a stock closing price at each one of these dates. I used the offset.Bday() method to account for any holidays falling on those prior and posterior dates.
The Bloomberg data was also merged to the main frame based on dates and tickers.

Then, I created the needed differenced columns. a value of 1 was given to a positive difference, denoting a buy recommendation, and a value of 0 was given to a negative difference, denoting a sell recommendation. The `.shift()` was used to difference rows from different columns. Given the classification nature of the exercise, once the binary columns were obtained, the original columns were dropped to unencumber the data frame.

## 3. Exploratory Data Analysis

### 3.1 Correlation
I analyzed the correlation between the features and the possible targets.

A pairplot did not help visualize any correlations mainly given to the fact that the data is differenced, and the features are binarized, taking away any information about a possible trend.

A heatmap was applied on all numerical data and , as expected, revealed, only strong correlations between the possible targets but no correlations between the possible target and the secondary variables. It is however, somewhat surprising that the variables studying the analysts' recommendations did not show any meaningful correlation with the target variables.

A meger correlation of (-.12) was found between `delta_3/7_np`(the direction of the stock until the end of the period) and `delta_1_0` (the price movement 1 day after the release) from the one hand and `delta_7_n0`  and `period`(the duration between two releases). The model will be tested with and without these independent variables added.

![correlation analysis](./images/correlation_better.png)

### 3.2 Clustering

In addition to feature engineering, I ran a K-Means clustering exercise on the different stocks. The logic is that, as in features correlation, some observation might be easier for the model to discern when they share some properties. For diversification purposes, the market invests in all sectors, and choose what it perceives are the best stocks, within that sector, based on a certain risk/return tradeoff.

These stocks came out as being in the same cluster `['aapl' 'amzn' 'csco' 'ebay' 'fcbk' 'msft']`
![clustering image](./images/cluster.png)

## VI. Model selection

I fitted several models to my data, including:
- BernoulliNB
- LogisticRegression
- KNeighborsClassifier
- DecisionTreeClassifier
- BaggingClassifier
- RandomForestClassifier
- ExtraTreesClassifier
- AdaBoostClassifier
- svm.SVC

Among these models only 2 models stood out by constantly beating the baseline prediction (the majority class). These models are : `BernoulliNB` and `SVM`. The `LogisticRegression` was a strong 3rd.

In terms of text vectorizers, the use of either the `TF-IDF` or the `CountVectorizer` yielded the same score.

### 1. Feature selection

As stated above my main feature is the text of the 8-k SEC filing.
Specifically, I trained my model on: `negative_text`, `positive_text`, and `uncertainty_text`. These text have been tried separately and in combination, as follows:
These features resulted in the following
- negative text
- positive text
- uncertainty text
- negative/positive
- negative/uncertainty
- positive/uncertainty
- negative/uncertainty/positive

The standalone `negative text` gave the best results.

In terms of additional (numerical) feature, the EDA suggested that there might be a benefit in adding `delta_1_0` `an_buy`, and `period` as secondary features. These have also been tried separately and in combination, but no added value came out of them. The model had `one feature` and `one target`.

### 2. Stocks/Cluster selection

The `clustering` exercise let emerge a particular cluster out of the whole group of stocks. The model scored best with that particular cluster of stocks than with all the stocks.

### 3. Dimensionality reducing algorithm

Given the large dimensionality created through the vectorization of the text, I thought of using Principal Components Analysis (PCA). However it is not compatible with the BernoulliNB because the PCA tend to put data in a '[-1,1]' range, whereas the BernoulliNB does not accept negative features.
The alternative was the TruncatedSVD, which is a PCA-like algorithm, known as LSA(latent semantic analysis). However, it did not improve the model.

## V. Model evaluation

### 1. Baseline prediction vs. Model Outcome

I selected `delta_7_np` as the model target(the stock direction between day 7 of the press release and the day prior to the next release). By a slight margin, it is the target that the model has the most success with predicting.
The majority class (`class_1`) representS 57.9% of the total outcomes. **57.9%** is, thus, my benchmark to evaluate any predictive value of my model.

The BernoulliNB model and the SVC models resulted in a Test score of **68.2%** and **66.6%**, respectively. Although, this result might not seem to be high, in a financial context, beating the market by an `alpha` of over 5% is considerable.

The model’s accuracy came higher than the base line prediction. Based on this outcome, and given these particular stocks, we could conclude that machines can be trained to analyze non-numerical financial information and avoid bias, when humans can’t.

### 2. Grid searching and Hyperparameter fine-tuning

Both the classifiers BNB and SVC and the vectorizers TF-IDF and CountVectorizer have been fine tuned. However, the default parameters yielded the best score.

### 3. Confusion Matrix

The confusion matrix shows a low specificity rate at 20% (compared to a baseline specificity rate of 0%). That is due to a high number of False positives: a buy recommendation, when the stock actually ends up trading lower. The model, however makes up the loss in specificity by a high sensitivity ratio of 92.75% (compared to the baselines' 100%). These are due to high number of True positives and low number of False negatives.
below is a depiction of the confusion matrix
![Confusion Matrix](./images/confusion_matrix.png)

### 3. Trading simulation

I simulated real transactions whereby I contrasted the results of trades out of the model's recommendations (`buy and sell`) and compared them to the market's `passive strategy` of just buying stocks. As suggested by the model's score, the simulation resulted in a higher return in absolute terms **8.5%** compared to the market's performance of **8.3%** over the observed period. However, this does not reflect the model's relatively much higher score (68% vs. 59%). This is mainly due to the binary nature of the model which is not sensitive to the magnitude of the increases and decreases in stock prices. It could well be that the model was more predictive than the market baseline, but it could also be that the market had more important wins and/or less important losses than the model's score suggests. the superiority of the model was, therefore, partially offset by the lack of granularity. This will certainly be the subject of future improvements of the model.
Below is a depiction of the Model's cumulative gains (and losses) compared to the market.
Please note that the higher line of the market does not mean that the market was superior. It's just that the market, by being always on the buy side, it invested more money and had higher return in absolute terms. However, the returns on investment were slightly higher for the model as explained above.
![Trading simulation](./images/market_vs_modelre.png)

## VI. Model Improvement

Future improvements to the model could be the following:
  - going from a binary model to multi-classification model to a regression model:
  - this model is considered to be too arbitrary. An increase or a decrease of, say, 5% had the same weight as an increase/decrease of half a percent.
  - The upgraded model should have a middle class whereby no trades are recommended within, say a bracket of [-2, 2]% to avoid unproportionately high transaction fees. 
  - We can increase the number of classes to cater for the magnitude of the increase or the decrease. The model would have a stronger impact when it caters for the amount of investment to the level of predicted movement, i.e. trade more when the gap is predicted high.
  - the same way the model was compared to the market, it would be interesting to see how it compares to analyst's predictions
  - Add social media feeds as a text feature to see their impact

#### Attachments:
- The code folder with all the notebooks organized along the processes described in this executive summary.
- The report folder with the executive summary and the accompanying images
- The sources folder with links to the different libraries used in this project
