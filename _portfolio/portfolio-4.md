---
title: "Exploring AWS/Lambda/EMR for Categorization Analysis"
excerpt: "Using Bayesian Logistic regression model in R programming language<br/><img src='/images/500x300.png'>"
collection: portfolio
---

Colin Fraser, Joseph Juhn (jjuhn2@illinois.edu), Julia Tcholakova

# Abstract
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