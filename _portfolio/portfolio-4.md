---
title: "Allocating Donations to the most suitable charity based on donor’s preferences In Canada"
excerpt: "Exploring AWS/Lambda/EMR for Categorization Analysis<br/><img src='/images/donate.jpeg'>"
collection: portfolio
---

Colin Fraser, Joseph Juhn (jjuhn2@illinois.edu), Julia Tcholakova

# ABSTRACT
This project has been developed in an effort to assist a charity platform in Vancouver, British Columbia. Our team’s intent is to help with allocating donations to the most suitable charity based on donor’s preferences. All charities in Canada have already been categorized by sector (program category) and each of them is listed by sector group. While this is helpful, the sector groups are broad and further details are needed in order for the charity platform to estimate proper allocation of donors’ contributions.
Our team has chosen to utilize Amazon Web Services with concentration on Lambda and MapReduce. For development tools we have used Python with PyCharm IDE. Our work includes the following: gathering the initial data for the top 110 charities in Canada, creating web scraping programs which produce the intermediate data used in further calculations, additional program which results in TF-IDF summary for each of the presented charities.

# INTRODUCTION AND MOTIVATION

There are over 80,000 charities in Canada, dedicating resources to a wide and varied range of charitable causes. Each year, Canadian charities submit a form to the Canadian Revenue Agency (CRA) to report contact information, financials, and a breakdown of the charitable causes that they support. Charities self-identify as contributing to one or more of about fifty program categories such as “Welfare of Domestic Animals”, “Disaster Relief”, etc. These labels are quite coarse, resulting in a high within-label variation of charitable activities. This can make it challenging for donors to find charities that contribute to the causes that they care about.
We are working to help out a charity platform in Vancouver, BC. Part of their motto is the following: “Our community connects people who want to make the world better with charities that are taking action on causes they care about”.
The charity platform works with thousands of charities and thousands of donors. It is not very easy to match manually donors with charities, unless the charity is specifically mentioned. It would be beneficial for the company to be able to group in some way all these charities and allocate the donations accordingly.

The main task of this project is to identify words and phrases that characterize charitable causes and activities.
These words and phrases can be used for tagging, categorization, and other related activities towards the goal of facilitating the discovery of charities. The application would read from text descriptions of the charity and output words and phrases that describe the charity’s causes and activities in more granular detail. For instance, from the CRA’s description of Alleycats Alliance Society1, we have:
Work with groups and individuals to advise or assist in the rescue of cats in need and other animal rescue, rehabilitate and put for adoption cats after vet care trap neuter and release cats after vet care”
While on the other hand, the description of Bird Ecology and Conservation Ontario reads :
“Research to advance the conservation of songbird species at risk in agricultural landscapes and research to improve our understanding of the distribution, abundance, and productivity of songbird species at risk.”
Each of these charities is classified as a “Protection of Animals” charity, but clearly from their descriptions they do different types of work. The application might extract words like “Rescue”, “Cats”, “Adoption”, or “Neuter” from the first description, whereas from the second it might extract words like “Research”, “Conservation”, or “Songbird”.
The words and phrases extracted from the descriptions can be used for a variety of applications including to create a more granular taxonomy of charities, as tags or search terms, as inputs into future classification projects or for topic modelling.


# APPLICATION AND ARCHITECTURE

This project divides into two parts. The first is the scraping phase, where we build the corpus of text that is used for the second part. The second part is to extract important words and phrases from the corpus generated in the first part. Both parts contain certain data cleaning and preprocessing. During the first phase the initial data is scraped and produced in a summary form, suitable for the next phase. During the second phase data, preprocessing includes activities such as tokenization, removal of punctuation and special characters, word stemming, removal of stopwords

![](.jpg)
Our main objectives for this project are first scalability, and second modularity. Our prototype version of the application only takes input from a few hundred charities, but the intent is to create an application which can easily be scaled to take data from tens of thousands or more. Both the scraper and the extraction of words and phrases take relatively general approaches, and the architecture would allow for either of these steps to be improved or extended easily. To accomplish this, we have made an extensive use of AWS cloud technologies—in particular, S3, Lambda and EMR. Details on how we have implemented this program with these technologies are given below.

# DEVELOPMENT PHASES
*A. Scraping*
The scraping phase consists of an application which takes one or more URLs, extracts text from the associated pages, and saves the text to json files.
We decided to keep the focus narrow for our scraper: we begin by scraping data from a single website that aggregates charity data. The website, charityintelligence.ca, maintains a list of reference pages of what it calls “4-star charities”, with the charity name, a text description of the charity, and a coarse category of charitable cause. This data is structured such that it is relatively straightforward to extract the categories and descriptions of each charity from the site. There are currently 110 4-star charities on the site, making this suitable as a proof- of-concept for the algorithm and application of cloud computing technologies. The modular design means that to change this to a more full-featured text processing tool we need only a small change of the scraper function – the TF-IDF calculation would be the same regardless of the scraper.
Our implementation of this task uses AWS Lambda to scrape the pages and saves the output to S3. We have completed the web scraping in two steps:
- We have built a link scraper program to extract the list of URLs for each of the top charities’ reference pages. The list is stored on Amazon S3.
- We have also built a scraper program, that takes the list of URLs, prepared in the previous step, scrapes the associated sites, and writes one file per scraped site in json format to a location on Amazon S3.
For each successfully scraped URL, the Lambda function writes a json file to an S3 bucket containing the name of the charity in question, charity sector, and the extracted text which is a summary description of the charity. Writing in this format (one output file per charity) is conducive to applying MapReduce applications in the next step.

Our strategy is to use Lambda to parallelize these tasks as much as possible. Each Lambda instance takes only a small number of URLs, and Lambdas can be triggered concurrently to provide scalability which is limited only by Amazon’s Lambda account-level concurrency limits (100 by default).

*Lambda Function Orchestration Strategies*
Some orchestration is required to orchestrate the individual lambda scrapers and point them to the correct targets. For this, we tested two strategies.
The first strategy was to have a persistent application orchestrating the Lambda functions. This orchestrating application would read the list of URLs, chunk them into small pieces, and send the chunks to the lambda functions This orchestrator could itself be implemented as a Lambda function, but this may lead into technical complications if the total time to scrape all of the URLs exceeds the Lambda execution time limit of five minutes. In practice, we expect that the orchestrator would be a small application that runs locally or could be deployed on an EC2 instance. The list of URLs for input is itself stored on S3 so that the orchestrator is not actually required to store that data. It is only concerned with chuncking the URLs and pointing the scrapers towards them. Thus, the orchestrator is not required to be a powerful machine or have much storage.

In practice, this strategy works quite well and is the strategy that we have adopted for our application.
However, we did experiment with another architecture which provides truly serverless orchestration. In the second strategy, the Lambda function takes index and chunk_size parameters. On launch, the Lambda reads the URL list and extracts the URLs from positions p=index to p=index+chunk_size. Then, before scraping, the Lambda asynchronously invokes a new Lambda function, setting the new index to index+chunk_size. By initializing this process with an initial index=0, a chain of Lambdas is automatically invoked to scrape the entire list.
A benefit of this architecture is: it dispenses with the need to have a persistent application monitoring and orchestrating, so in theory orchestration becomes very cheap and simple. However, special care is required to deal with failure handling and logging. Complications arise when one of the Lambda functions raises an exception or times out.
In practice, we found that the first approach is simpler to manage, and since the technical requirements of the machine hosting the persistent application are quite light, we found that this is the most effective approach for now.

We have experimented with different levels of text preprocessing during this phase and we have decided that all complex data transformations are to be done in the next phase. One of the main reasons for this is the fact that additional transformations at this step increase the runtime of the Lambda function itself, which increases the cost of each scrape—possibly by a significant amount, depending on the volume of URLs needed to be scraped.

*B. Extraction of words and phrases*
Our strategy for extracting relevant words and phrases from the corpus is to use Text Frequency-Inverse Document Frequency (TF-IDF) weighting on N-grams from the corpus of text generated by the first stage. TF-IDF weights are commonly used in information retrieval and text processing for scoring the importance of terms relative to documents. There are many versions of TF-IDF weighting, but the simplest is given as follows: Given a corpus of 𝑁documents, a word (or more generally a “term”) 𝑤, and a document 𝑑, we define the term frequency 𝑓 (𝑤, 𝑑) to be the number of times that 𝑤 occurs in 𝑑, and the inverse document frequency 𝑖𝑑𝑓(𝑤) to be 𝑁 divided by the number of documents containing 𝑤. Then, the TF-IDF score of 𝑤 in document 𝑑 is given as 𝑆(𝑤, 𝑑) = 𝑙𝑜𝑔(1 + 𝑓(𝑤, 𝑑)) × 𝑙𝑜𝑔(1 + 𝑖𝑑𝑓(𝑤)).
The TF-IDF weight has a number of attractive properties as a scoring function for information retrieval.
- A term that does not ever appear in any document is always given a score of 0.
- A term that appears in every document is always given a score of 0.
- A term 𝑤 that appears often in 𝑑1 but rarely in 𝑑2 will have 𝑆(𝑤, 𝑑1) > 𝑆(𝑤, 𝑑2).
- A term that occurs in only a few documents will have a higher score than a term appearing in many documents.

The TF-IDF weighting scheme is sensitive to how we decide what to call a term, a document, and a corpus. The most straightforward approach that we have used for our project is to compute TF-IDF weights with the entire output of the scraping phase as the corpus, each file as a document, and single words as terms, but any of these parameters could be changed to obtain different and potentially helpful results for later iterations of the program. Rather than looking at individual words as terms, we could use N-grams to find important phrases. We could limit the scope of a corpus to be only within a cause category, which would identify words which differentiate charities within the same category (e.g. differentiating Cat Shelters from Songbird Conservation Research funds).
Alternatively, we could consider cause categories to be documents, which would help us differentiate between categories rather than within them.
There are also extensions of the TF-IDF weighting scheme which address certain less desirable features of the simple TF- IDF weight. For instance, TF-IDF naturally favors longer documents over shorter ones—this may not be desirable behaviour and there have been methods proposed in the literature to mitigate this.
Our main goal for this project has been to implement simple TF-IDF weighting as defined above, with the natural definitions of terms, documents, and corpus.
The TF-IDF weighting has been implemented using MapReduce on Amazon Elastic MapReduce (EMR).

The full term frequency portion is given in pseudocode below

```python
def tf_wordcount_mapper(input): 
    for word in input:
        emit (word, document), 1
def tf_wordcount_reducer(key, values): 
    emit key, sum(values)
def tf_transform_mapper(input): 
    emit input
def tf_transform_reducer(key, value): 
    emit key, log(value + 1)
```
For our TF-IDF development we have utilized Python package, called ‘mrjob’, as it makes it easier to transition between local and cloud cluster modes. Mrjob allows developers to program locally and run the code on a cluster, regardless of the location of the cluster. The package has excellent integration with AWS EMR and that is the main reason our team has chosen to work with it.
To run our TF-IDF program, we have opted to work on Amazon EMR cluster with the following specs:

*Core Hadoop:*
 Hadoop 2.8.3 with Ganglia 3.7.2, Hive 2.3.2, Hue 4.1.0,
 Mahout 0.13.0, Pig 0.17.0, and Tez 0.8.
 
*Intermediate MR results:*
["kids", "Farm Radio International"] 0.000991626867753856 ["knowledge", "Farm Radio International"]
0.001983253735507712
["landline", "Farm Radio International"]
0.000991626867753856
["landlord", "Aunt Leah's"] 0.0009547481825894563 ["leads", "Farm Radio International"] 0.000991626867753856 ["leah", "Aunt Leah's"] 0.014321222738841845
["learn", "Aunt Leah's"] 0.0
["learn", "Farm Radio International"] 0.0
["literacy", "Farm Radio International"]
0.000991626867753856

The final output of the extraction phase would be an inverted index data structure where for each word there is an entry for each document that it appears with its TF-IDF score for that row as in the table below.

# Table 

# CONCLUSION
We are satisfied with the outcome of our project. We have created a solid simple tool for the charity platform, which they can further develop, expand its purposes and scale up easily in their research. We have done extensive testing with the Lambda functions and we are currently working to provide a separate methodology report for the charity platform. This report would describe the detailed steps needed for successful implementation and launching of AWS Lambda functions in the specific network environment of this company.

In addition, in our development and implementation we have used mainly open source products which could be easily maintained and integrated with other open source technologies, especially diverse cloud technologies.

# REFERENCES
[1] Roy Campbell, Reza Farivar “Cloud Computing Applications” (2018) Lectures
[2] AWS documentation (2018) [Online]. Available:
https://aws.amazon.com/documentation/?nc2=h_ql_d&awsm=ql-5
[3] Mrjob documentation (2018) [Online]. Available: https://pythonhosted.org/mrjob/
[4] Apache Hadoop documentation (2018) [Online]. Available: http://hadoop.apache.org/
[5] Charity Intelligence Canada (2018) [Online]. Available: https://charityintelligence.ca
