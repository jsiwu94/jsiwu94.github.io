---
layout: post
title: "Subreddit User Recommendation System based on Netflix's Algorithm"
date: 2019-12-17
excerpt: "Using SVD and Matrix Factorization to Recommend Subreddit Topics for Reddit User"
tags: [Machine Learning, Collaborative Filtering, Predictive Modelling, Factorization Matrix, Gradient Descent Algorithm ,SVD, Python]
comments: true
---

<img width="165" alt="Screen Shot 2019-12-17 at 10 50 28 PM" src="https://user-images.githubusercontent.com/54050356/71062990-a6d70180-2120-11ea-955e-90f677d4a721.png">
<p align="center">
  you can find my wordcloud tutorial here:
  <a href="https://github.com/jsiwu94/SVD_for_Subreddit_Recommendation/blob/master/redditbot_wordcloud.ipynb">Link</a>
  <br><br>
</p>


User recommendation system is widely used in a lot of e-commerce or entertainment businesses to encourage more customer satisfaction and generate revenue for the company. The most commonly known method for this is called the **collaborative filtering**. Companies such as Netflix, for example, uses this method to recommend movies that matches their users' taste. Inspired by Nexflix's world class user recommendation algorithm, I am going to generate a recommendation system to help [Reddit](https://www.reddit.com/) users find subreddit that they might enjoy in this project. [Find my full python code here](https://github.com/jsiwu94/SVD_for_Subreddit_Recommendation/blob/master/Subreddit_Recommendation_System.ipynb)


## Introduction

Before we began, let me first introduce you to reddit. What is Reddit? Reddit is a website which provide users with news aggregation, web content rating, and discussion. In addition, it allows its members to submit content to the site such as text, images, or links, which other members can interact with by leaving comments or voting up or down. One main component of Reddit is the **subreddit**. Subreddits are specific online communities within the Reddit website that serve as a topic or category that organize all the posts associated with them. 

Despite being a very popular website with over 300 million users based on the data collected in 2018, there is still no good way to discover subreddits. Currently, users have to manually search for popular subreddits or upvote a subreddit so that it will be archived in their subreddit history.

## Getting The Data

To get the dataset, I utilized the Reddit API for python called [PRAW](https://praw.readthedocs.io/en/latest/code_overview/models/submission.html). The main requirements are to have your client_id, client_secret, user_agent. Using the API, I collected a dataset consisting  of ~15K unique redditors (reddit users) and ~29K unique subreddits in the below format: 
- the redditors' username
- their subreddit submissions (or comments)
- the timestamp (utc) when they create the submission

Because it took a long time to scrape the data, I exported the final output to a csv. [My PRAW Code](https://github.com/jsiwu94/SVD_for_Subreddit_Recommendation/blob/master/PRAW.ipynb) is also available on my GitHub.

```python
reddit = praw.Reddit(client_id='your client id',
                    client_secret='your client secret',
                    user_agent='project name')

r = reddit.subreddit('all').top(limit=15000)
redditor = []
subreddit_name = []
utc = []
i=0

for submission in r:
    redditor.append(submission.author)
    subreddit_name.append(submission.subreddit)
    utc.append(submission.created_utc)
    i+=1
    
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>username</th>
      <th>subreddit</th>
      <th>utc</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>kabanossi</td>
      <td>photoshopbattles</td>
      <td>1.482748e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>kabanossi</td>
      <td>GetMotivated</td>
      <td>1.482748e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>kabanossi</td>
      <td>vmware</td>
      <td>1.482748e+09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>kabanossi</td>
      <td>carporn</td>
      <td>1.482748e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>kabanossi</td>
      <td>DIY</td>
      <td>1.482747e+09</td>
    </tr>
  </tbody>
</table>

**Checking the Data**
The Data contains 15K redditors and ~29K subreddits.

    unique reddittor: 15000
    unique subreddit: 29281
    total data entry: (9391244, 3)
    
    Are there null values from our API dataset?  
    username     False
    subreddit    False
    utc          False
    dtype: bool

## Collaborative Filtering and Singular Value Decomposition 

Now that we have the dataset, let's start building the algorithm. For this project, I used the Singular Value Decomposition (SVD) technique to create the subreddit recommendation. 

The SVD is a well-known matrix factorization method and is very well-studied. The winning team at the [Netflix Prize competition in 2009](https://medium.com/netflix-techblog/netflix-recommendations-beyond-the-5-stars-part-1-55838468f429) used some combinations of advanced SVD matrix factorization models to produce movie recommendations with an improved RMSE of ~8% from Netflix's Recommendation System at the time. 

Before we begin, let's look into the concept behind it first. The **Collaborative filtering** is a method to predict a rating for a user item pair based on the history of ratings given by the user and given to the item. 

<i>"Collaborative filtering captures the underlying pattern of interests of like-minded users and uses the choices and preferences of similar users to suggest new items."</i>

**SVD** is a matrix factorization technique that is used to reduce the number of features of a data set by reducing space dimensions. The matrix factorization is done on the user-item ratings matrix. From a high level, matrix factorization can be thought of as (using product and factorization) to find 2 matrices whose product is the original matrix. In other words, it approximates a single matrix A by the product of three matrices.

<img width="676" alt="Screen Shot 2019-12-17 at 11 49 17 PM" src="https://user-images.githubusercontent.com/54050356/71066648-e3f2c200-2127-11ea-96a1-143a90fbce7e.png">

In our context, U contains data for each username and their subreddit submissions, V is the subreddit name and its number of submissions by the user, while Σ are called the singular values and it indicates the strength of how U and V are related.
We can approximate the full matrix by observing only the **most important features** using the singular values (Σ). To read more about SVD please refer to these <b>research papers:</b>
[1] <a href="http://buzzard.ups.edu/courses/2014spring/420projects/math420-UPS-spring-2014-gower-netflix-SVD.pdf">Netflix SVD</a> [2] <a href="https://www.cs.uic.edu/~liub/KDD-cup-2007/proceedings/Regular-Paterek.pdf
">SVD for Recommendation Engine</a> 

Now that you have enough theoritical information about SVD, let's look into the application. In this project, I will be using python's scipy package of [sparsesvd](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.linalg.svds.html). With this package, the requirement is to have our data in a matrix format and state the **latent factor** or the number of important features we would like to consider. I will explain more about this in the later section.

#### Libraries
Below are the libraries I used:

```python
import numpy as np
import pandas as pd
import math as mt
import csv
from pandas import DataFrame,Series,read_csv
import scipy
import scipy.sparse as sp
from sparsesvd import sparsesvd        #used for matrix factorization
from scipy.sparse import csc_matrix    #used for sparse matrix
from scipy.sparse.linalg import *      #used for matrix multiplication
from scipy.linalg import sqrtm
from nltk.tokenize import TreebankWordTokenizer #used this to create the document on my matrix
```


## Evaluating our SVD Model with Train and Test Data by Sampling 500 users

Using a sample of 500 users from the entire dataset, I created both train and test data (20% the data in the test data) to evaluate the prediction of my SVD model. 

```python
sample_username = list(reddit_df.username.unique())[300:800]
sample_df = reddit_df[reddit_df.username.isin(sample_username)]

users = list(sample_df.username.unique())
subreddits = list(sample_df.subreddit.unique())
```

Let's look at the top 65% subreddits within the sample dataset. There are ~134 subreddits that contribute to the entire sample dataset. We will use this for our **latent factor** in the SVD model later.

    Top 134 subreddits contribute a total of 64.9 % to the total subreddits in the dataset
    
#### Splitting Train and Test Dataset based on Timestamp (utc)

I wanted to ensure that my test and train data are as realistic as possible. Therefore, I used the timestamp (utc) information to split the data instead of doing random 20% split. In this case, I sorted the data from the earliest submission to the most recent by each user, and using those, pick the bottom or most recent 20% data of each user as the test data.

```python
data =sample_df.groupby(['username','subreddit']).agg({'subreddit':'count',
                                                                 'utc':'max'}).\
              rename(columns={'subreddit':'submission_freq','utc':'most_recent_timestamp'}).reset_index()
data.head(10)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>username</th>
      <th>subreddit</th>
      <th>submission_freq</th>
      <th>most_recent_timestamp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-SA-HatfulOfHollow</td>
      <td>news</td>
      <td>1</td>
      <td>1.482761e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-SA-HatfulOfHollow</td>
      <td>reddevils</td>
      <td>1</td>
      <td>1.482742e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-SA-HatfulOfHollow</td>
      <td>soccer</td>
      <td>1</td>
      <td>1.482771e+09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-SA-HatfulOfHollow</td>
      <td>worldnews</td>
      <td>11</td>
      <td>1.476293e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-_-_-_-otalp-_-_-_-</td>
      <td>Android</td>
      <td>3</td>
      <td>1.475605e+09</td>
    </tr>
    <tr>
      <th>5</th>
      <td>-_-_-_-otalp-_-_-_-</td>
      <td>AskAnthropology</td>
      <td>2</td>
      <td>1.480134e+09</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-_-_-_-otalp-_-_-_-</td>
      <td>AskReddit</td>
      <td>2</td>
      <td>1.482744e+09</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-_-_-_-otalp-_-_-_-</td>
      <td>BlackPeopleTwitter</td>
      <td>6</td>
      <td>1.482560e+09</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-_-_-_-otalp-_-_-_-</td>
      <td>CrazyIdeas</td>
      <td>1</td>
      <td>1.480079e+09</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-_-_-_-otalp-_-_-_-</td>
      <td>DC_Cinematic</td>
      <td>2</td>
      <td>1.476638e+09</td>
    </tr>
  </tbody>
</table>


#### Defining The Rating in The Train and Test Data

Evaluating our SVD model means that we are comparing the predicted vs actual **rating** that a user give to a subreddit. Since there is no obvious way to capture rating with Reddit data, we will define the rating as the number of times user i has submmited a post in subreddit j, vi,j, divided by his total number of submission.

<img width="169" alt="Screen Shot 2019-12-18 at 1 00 41 PM" src="https://user-images.githubusercontent.com/54050356/71122933-78940900-2196-11ea-9cfc-5c3c2b365983.png">

In a different studies by [Jay Baxter](http://jaybaxter.net/redditrecommender.pdf), a user upvote can also be used to capture the rating. However, the user upvote data are proctected and can not be retrieved using PRAW (Reddit API for Python). Therefore, I used user submission instead.

```python
user_sum = data.groupby(['username'], as_index=False).agg({'submission_freq':'sum'})
temp = pd.merge(left = data, right = user_sum, how='left', left_on='username',right_on='username').\
                rename(columns={'submission_freq_y':'user_sum',
                               'submission_freq_x':'submission_freq'})
data['user_implicit_rating'] = temp.submission_freq/temp.user_sum
data.drop(columns=['submission_freq'], inplace=True)
data = pd.concat([data.iloc[:,:2],data.iloc[:,-1:],data['most_recent_timestamp']], axis=1)
data.dropna(inplace = True)
```

 Let's look at an example train and test data for a user.

    Train Data for User "-_-_-_-otalp-_-_-_-"        :
                      username             subreddit  user_implicit_rating
    11220  -_-_-_-otalp-_-_-_-             worldnews              0.001017
    11222  -_-_-_-otalp-_-_-_-            OCOCTATIAT              0.001017
    11365  -_-_-_-otalp-_-_-_-           nottheonion              0.001017
    11405  -_-_-_-otalp-_-_-_-     millionairemakers              0.001017
    11600  -_-_-_-otalp-_-_-_-          changemyview              0.001017
    11650  -_-_-_-otalp-_-_-_-                  news              0.001017
    12684  -_-_-_-otalp-_-_-_-               Android              0.003052
    13535  -_-_-_-otalp-_-_-_-               chomsky              0.002035
    13634  -_-_-_-otalp-_-_-_-          DC_Cinematic              0.002035
    14440  -_-_-_-otalp-_-_-_-        TheoryOfReddit              0.002035
    14453  -_-_-_-otalp-_-_-_-         UpliftingNews              0.001017
    14629  -_-_-_-otalp-_-_-_-          the_meltdown              0.001017
    15198  -_-_-_-otalp-_-_-_-               pokemon              0.001017
    15316  -_-_-_-otalp-_-_-_-             australia              0.009156
    15339  -_-_-_-otalp-_-_-_-  Political_Revolution              0.005086
    15684  -_-_-_-otalp-_-_-_-                 funny              0.004069
    15685  -_-_-_-otalp-_-_-_-                 Jokes              0.001017
    16391  -_-_-_-otalp-_-_-_-                  gifs              0.001017
    16633  -_-_-_-otalp-_-_-_-              counting              0.001017
    17434  -_-_-_-otalp-_-_-_-                iphone              0.004069
    19305  -_-_-_-otalp-_-_-_-            CrazyIdeas              0.001017
    19452  -_-_-_-otalp-_-_-_-            badhistory              0.001017
    19464  -_-_-_-otalp-_-_-_-       AskAnthropology              0.002035
    21413  -_-_-_-otalp-_-_-_-                 otalp              0.001017
    23433  -_-_-_-otalp-_-_-_-       depressedrobots              0.001017
    23808  -_-_-_-otalp-_-_-_-       EnoughTrumpSpam              0.053917
     
    Test Data for User "-_-_-_-otalp-_-_-_-"        :
                      username           subreddit  user_implicit_rating
    25662  -_-_-_-otalp-_-_-_-              me_irl              0.001017
    25708  -_-_-_-otalp-_-_-_-           socialism              0.002035
    26122  -_-_-_-otalp-_-_-_-           spacemacs              0.001017
    26438  -_-_-_-otalp-_-_-_-           radiohead              0.008138
    26816  -_-_-_-otalp-_-_-_-               meirl              0.001017
    27038  -_-_-_-otalp-_-_-_-               Music              0.005086
    27570  -_-_-_-otalp-_-_-_-          indieheads              0.001017
    28582  -_-_-_-otalp-_-_-_-      Showerthoughts              0.003052
    28839  -_-_-_-otalp-_-_-_-              movies              0.014242
    29181  -_-_-_-otalp-_-_-_-            politics              0.060020
    29186  -_-_-_-otalp-_-_-_-  BlackPeopleTwitter              0.006104
    29432  -_-_-_-otalp-_-_-_-           dagandred              0.010173
    29917  -_-_-_-otalp-_-_-_-         LiverpoolFC              0.379451
    29974  -_-_-_-otalp-_-_-_-            Fuck2016              0.018311
    30335  -_-_-_-otalp-_-_-_-            bidenbro              0.001017
    30379  -_-_-_-otalp-_-_-_-               apple              0.005086
    30955  -_-_-_-otalp-_-_-_-             atheism              0.001017
    30956  -_-_-_-otalp-_-_-_-           AskReddit              0.002035
    30969  -_-_-_-otalp-_-_-_-          botsrights              0.027467
    31486  -_-_-_-otalp-_-_-_-              soccer              0.348932


## Transforming the Dataframe into Utility Matrix and SVD Computation

As mentioned earlier, one of the requirement to use SVD model is to transfor our data into a matrix. In this case, I will fill all the subreddits that a user has not had submission on with zero's. Once we have our matrix ready, let's create the SVD function.

```python
def svd(train, k):
    utilMat = np.array(train)
    # the nan or unavailable entries are masked
    mask = np.isnan(utilMat)
    masked_arr = np.ma.masked_array(utilMat, mask)
    item_means = np.mean(masked_arr, axis=0)
    # nan entries will replaced by the average rating for each item
    utilMat = masked_arr.filled(item_means)
    x = np.tile(item_means, (utilMat.shape[0],1))
    # we remove the per item average from all entries.
    # the above mentioned nan entries will be essentially zero now
    utilMat = utilMat - x
    # The magic happens here. U and V are user and item features
    U, s, V=np.linalg.svd(utilMat, full_matrices=False)
    s=np.diag(s)
    # we take only the k most significant features
    s=s[0:k,0:k]
    U=U[:,0:k]
    V=V[0:k,:]
    s_root=sqrtm(s)
    Usk=np.dot(U,s_root)
    skV=np.dot(s_root,V)
    UsV = np.dot(Usk, skV)
    UsV = UsV + x
    print("svd done")
    return UsV
```

Now let's run our prediction on test data and compare it againts the actual. In this case, I will use MSE and MAE.

```python
def mse(true, pred):
    # this will be used towards the end
    x = true - pred
    return sum([xi*xi for xi in x])/len(x)

def mae(true, pred):
    # this will be used towards the end
    x = abs(true - pred)
    return sum([xi for xi in x])/len(x)


# to test the performance over a different number of features
no_of_features = [134]
svdout = svd(X, k=no_of_features)

print(mse(test['user_implicit_rating'], pred))
print(mae(test['user_implicit_rating'], pred))
```

Based on the output, we got a pretty good result with a MAE is around 0.5%. 

    svd done
    mse: 0.0001907806366327251
    mae: 0.005219784277697061


## Creating The Recommendation System Using The Complete Dataset (15K Users)

Now that we have evaluated the model, let's use the entire dataset and make a demo recommendation. To do this, we will do the same steps of converting the data into a matrix as well as finding the latent factor.

In the full dataset, there are 293 subreddits contributing to the top 65% subreddit topics. Therefore, I used this number to define the latent factor in the SVD model.

    Top 293 subreddits contribute a total of 65.0 % to the total subreddits in the dataset


Given that this dataset is much larger than the sample, we will use another method to organize the data and alter it into a matrix form for the SVD. 

In this case, I changed the row in the dataset to be 1 row per each user and tokenize all the subreddits where each users had posted a submission on using the TreebankWordTokenizer from the [nltk](https://www.nltk.org/) library.

```python
tokenizer = TreebankWordTokenizer()
document = doc_df.iloc[:, 1]
document = document.apply(lambda row: tokenizer.tokenize(row))
document.head()
```
    0    [Testosterone, Testosterone, Testosterone, Tes...
    1    [DestinyTheGame, DestinyTheGame, DestinyTheGam...
    2    [AceAttorney, AceAttorney, AceAttorney, AceAtt...
    3    [LGBTeens, Patriots, asktransgender, Patriots,...
    4    [tdi, tdi, tdi, AskReddit, tdi, tdi, tdi, tdi,...
    Name: subreddit, dtype: object



## Creating User-Subreddit Matrix

Once we tokenized the subreddit, time to create our matrix for the SVD. Our matrix is very sparse in that it has a lot of zeros because there are a lot of user-subreddit combinations where the users have not posted a submission on. This large and sparse matrix can take a long time to run. Therefore, I will use the [sparse matrix](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csc_matrix.html) from the scipy library. This helps storing the matrices in less memory.

```python
corpus_of_subs = []
for subreddits in subreddit:
    corpus_of_subs.append(subreddits)


voc2id = dict(zip(corpus_of_subs, range(len(corpus_of_subs))))
rows, cols, vals = [], [], []
for r, d in enumerate(document):
    for e in d:
        if voc2id.get(e) is not None:
            rows.append(r)
            cols.append(voc2id[e])
            vals.append(1)
user_subreddit_matrix = csc_matrix((vals, (rows, cols)), dtype=np.float32)
print((user_subreddit_matrix.shape))
```
    (14999, 29280)


Now, our matrix is pretty much ready and we run the SVD and the recommendation with the full matrix. 

#### SVD Function
```python
def computeSVD(user_subreddit_matrix, no_of_latent_factors):
    
    """Compute the SVD of the given matrix.
    :user_subreddit_matrix: a numeric matrix
    :no_of_latent_factors : numeric scalar value
    
    :U  : User to concept matrix 
    :S  : Strength of the concepts matrix
    :Vt : Subreddit to concept matrix
    """
    U, s, Vt = sparsesvd(user_subreddit_matrix, no_of_latent_factors)
    
    dim = (len(s), len(s))
    S = np.zeros(dim, dtype=np.float32)
    for i in range(0, len(s)):
        S[i,i] = mt.sqrt(s[i])

    U = csc_matrix(np.transpose(U), dtype=np.float32)
    S = csc_matrix(S, dtype=np.float32)
    Vt = csc_matrix(Vt, dtype=np.float32)

    return U, S, Vt
```

#### Recommendation Output Function
```python
#Compute estimated recommendations for the given user
def computeEstimatedRecommendation(U, S, Vt, uTest):
    """Compute the recommendation for the given user.
    
    :U     : User to concept matrix 
    :S     : Strength of the concepts matrix
    :Vt    : Subreddit to concept matrix
    :uTest : Index of the user for which the recommendation has to be made
    
    :recom : List of recommendations made to the user
    """
 
    #constants defining the dimensions of the estimated rating matrix
    MAX_PID = len(subreddit)
    MAX_UID = len(user)
    
    rightTerm = S*Vt 

    EstimatedRecommendation = np.zeros(shape=(MAX_UID, MAX_PID), dtype=np.float16)
    for userTest in uTest:
        prod = U[userTest, :]*rightTerm
        # Converting the vector to dense format in order to get the indices 
        # of the movies with the best estimated ratings 
        
        EstimatedRecommendation[userTest, :] = prod.todense()
        recom = (-EstimatedRecommendation[userTest, :]).argsort()[:293]
    return recom
```

Below are some recommendation demo based on the output of our SVD model using the entire dataset of 15K users and ~29 subreddits.

## Recommendation Demo 1


```python
uTest = [np.where(user == 'CarnationsPls')[0][0]]
U, S, Vt = computeSVD(user_subreddit_matrix, no_of_latent_factors)
```
    ------------------------------------------------------------------------------------
    
    Redditor: CarnationsPls
    
    ------------------------------------------------------------------------------------
    
    User Subreddit History :
    
    sports
    gaming
    gifs
    AskReddit
    fo4
    todayilearned
    TheLastAirbender
    realrule34
    ------------------------------------------------------------------------------------
    
    Recommendation for CarnationsPls : 
    
    Calgary
    Amd
    pcgaming
    techsupport
    NoMansSkyTheGame
    ------------------------------------------------------------------------------------
    


## Recommendation Demo 2

```python
uTest = [np.where(user == 'comicfan815')[0][0]]
U, S, Vt = computeSVD(user_subreddit_matrix, no_of_latent_factors)
```

    ------------------------------------------------------------------------------------
    
    Redditor: comicfan815
    
    ------------------------------------------------------------------------------------
    
    User Subreddit History :
    
    reactiongifs
    cringepics
    nba
    lakers
    NBA2k
    ------------------------------------------------------------------------------------
    
    Recommendation for comicfan815 : 
    
    rockets
    warriors
    bostonceltics
    sixers
    torontoraptors
    ------------------------------------------------------------------------------------
    
## Conclusion

SVD is definitely a great technique for collaborative filtering and it handles large dataset really well in that it is able to sort through the most important features to use for the prediction. There are so much more things that we can do with SVD to create better prediction for collaborative filtering, including utilizing gradient descent to minimize the error produced by the SVD model. While this project is only a POC (Prove of Concept) and I am only using the simple built-in SVD function by the scipy library, I hope to utilize this method with the conjuction of other technique such as the Restricted Boltzman Machine (RBM) in my future work.

## References
Hutchinson, A. (2018, April 20). Reddit Now Has as Many Users as Twitter, and Far Higher Engagement Rates. Retrieved from https://www.socialmediatoday.com/news/reddit-now-has-as-many-users-as-twitter-and-far-higher-engagement-rates/521789/.

Baxter, J. (2016). A comparative analysis of subreddit recommenders for Reddit. Retrieved from http://jaybaxter.net/redditrecommender.pdf

Salakhutdinov, R. & Mnih, A. & Hinton G. (2007). Restricted Boltzmann Machines for Collaborative Filtering. Retrieved from 
https://www.cs.toronto.edu/~rsalakhu/papers/rbmcf.pdf




**Thank you for reading! Please feel free to contact me directly for any comments, feedbacks, or suggestions. You can leave a comment below as well!**

Analysis and Report Written by : Jennifer Siwu 
