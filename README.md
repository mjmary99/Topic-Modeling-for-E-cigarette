# Topic-Modeling-for-E-cigarette

Electronic cigarettes were invented in 2003 by a Chinese Pharmacist. It aims to become a healthier alternative for regular tobacco cigarettes. After entering the US market in 2007, the American legislations committee then regulates this product during the hearings due to the lack of relevant laws. This project will focus on the hearings from 2008-2017 on topic changes for the e-cigarettes area.

This project will first transform the raw video/audio files into text files. After fixing the parameters of the model by comparing the auto-transcriptions and the accurate human transcribed version, the model will be applied to all videos for generating auto-transcriptions. Then, the project will identify the specific portion of the text files related to electronic cigarettes, and model the topic changes trends using topic models given specific covariates including the time, and the party in control.

## Files included:

#### File #1: extract_full_youtube_transcription.ipynb
This is the file to generate the video id, state, date, and full transcript from Youtube API. It will take a long time to run the code due to the length and the amount of the videos, so I put every generated data files (partial data because the original file exceeds 20M) inside already. There's no need to run this file, and the final result for this file is: id_state_date_full_final.csv.

#### File #2: extract_portion_youtube_transcription.ipynb
This is the file to generate the video id, state, date, and portion transcript from Youtube API. Instead of generating the full transcript, it only keeps the portion that relates with our topic. This file takes some time to run as well. The generated final result from this result is: id_state_date_portion_final.csv.

#### File #3: google_transcript.py
This file is to extract the transcript from Google Speech-to-Text API. Inside the file, it uses an example of one of the videos we uploaded to Google Cloud.

#### File #4: word_error_rate.py
This file is to calculate the word error rate (WER) between two paragraphs/sentences. In the last line, it includes two simple sentences just as an example due to the length of our actual texts. If want to change the texting content, just need to change the contents in the last printing command.

#### File #5: meta_data_cleaning.py
Due to the difficulty in reading dates from the first two files, this file will get the dates and transform the dates into consecutive natural numbers. Other elements/metadata are already been generated inside the first two files.

#### File #6: finalizing_data
This is an SQL file to clean the invalid data, eliminat empty cells, and combine the above data into one CSV file.

#### File #7: modeling.rmd
This is the modeling file using R. We used the structual topic modeling (stm) library inside R to build our model. Along with multiple visualizations inside the file. This file also takes a long time to generate graphs due to the amount of the words inside our database.

#### Data Folder
There are four datasets inside this folder. All of them are top 30 sample data from the whole datasets due to the exceeding of the upper limit.


## Data Sets
In this project, we will have three sources of data: built-in transcripts from the Youtube link using Youtube API, auto-transcripts using Google Speech-to-Text API, and human transcriptions. Inside the Youtube Channel, we have 405 videos with their built-in transcripts. And, we uploaded the same 405 videos to Google cloud for generating auto-transcripts. Meanwhile, we have 15 human transcripts that are the specific portion related to the E-Cigarettes topic. We are going to calculate the word error rate between human transcripts and Youtube transcripts, and human transcripts and Google Speech-to-Text transcripts.

Aside from getting the transcription based on Youtube’s built-in transcribing function, we also extracted the name and the descriptions using the youtube API to get the metadata variables for each video: state and date. For each video, the video’s name and description shows the date and the state of each video in the youtube channel. In order to better analyze the trends, we created a new column for day in correspondence to the date column.
<p align="center">
<img width="695" alt="Picture1" src="https://user-images.githubusercontent.com/61670089/222326358-604d4538-a551-4261-9470-b4fefaf74ce5.png">
</p>

## Overall Technical Approach

#### Transcriptions
As the human transcriptions of some of the hearings are provided by Professor Boushey, we only need to derive the auto-transcriptions by ourselves. Inside the Youtube channel, we have a total of 442 videos. After extracting the duplicated videos and the ones that didn’t enable auto-transcriptions, we have 405 videos left. To achieve the full transcripts of these videos, we used both Youtube API and Google API to batch transcript the 405 hearings. From Youtube API, we got the auto-transcriptions with the default parameters, while the parameters such as number of speakers, rate hertz, language code, etc are being assigned manually by us in Google API. The data sets we got from are shown above in the “data sets” section.

#### Locating the E-cig related portion by Keyword Analysis
To locate the portions of the hearings that are discussing electronic cigarettes, we separate each of the transcripts into multiple segments with 300 words each. Inside each segment, we calculated the sum of the keyword frequency based on the list provided by Professor Boushey: ‘tobacco’, ‘nicotine’, ‘cig’, ‘smok’, ‘vap’, ‘eliquid’/’e-liquid’, ‘ejuice’/’e-juice’. Some of the words in this list aren't the full word because sometimes they might have different versions of the word, so we can only take the root words. After getting the frequencies for different segments, we will only extract the parts that have multiple keywords. To be qualified as multiple keywords, sum(keywords inside the intended segment + keywords inside the segment before + keywords inside the segment after) needs to be larger than 1% of the total keywords inside the whole transcript. In order to get the idea of the surrounding contents, we also saved 1 segment before and 1 segment after the intended portion. By doing so, it became possible for us to locate the e-cig related portions of transcriptions and use them for our structural topic modeling.

#### Structural Topic Modeling

Structural Topic Modeling is an unsupervised machine learning model. It has the implementation on R that is named “stm” package. This model is designed to help users to figure out the topics and evaluate their relationships to the metadata. With this model, we are able to do machine-assisted reading of the large scale of text data at document level for finding topics that are related to electronic cigarettes. It can even discover the relevant topics that may not be recognized by our human beings.

Metadata serves as an important part for drawing conclusions when defining the relationship between topics. In our model, we include the metadata like “State”, and “Date” since we want to measure the change of topics when related to various parties, states, or timelines. The results of trends seem to be dramatically different as those metadata are included.

The input for this model in R will be a CSV that has multiple rows of document information. We cleaned and processed the data we had retrieved previously before directly putting them into the model. We extracted the date and region for each video from their name as we previously did through using Youtube API. The transcripts part is the combination of the string text data of the E-cig related portion by keyword analysis in each transcript. After these preparations, the dataframe we created using elements above was exported to a CSV through using numpy.

The output of this model will be sets of topics in portions as shown below. The 19th topic set seems to be the topics that we most likely want based on our assumptions. They are all related to e-cigarette. For other topics set, we can also see a high frequency of “bill” in each topic which is also related to our keywords of e-cigarette.

The parameters such as “K”, the number of topics we want the model to extract is determined by a series of experiments and evaluations which will be deliberately explained in section 6. For now, we are just further construct a topic model with the e-cig portion that we extracted before from keyword analysis, and the new parameter “K” is  15 as it was the best number by our evaluation. Here are some visualizations of our e-cig portion’s model from different perspectives and dimensions. (Start from here are some new progress we made after second draft of report)
<p align="center">
<img width="401" alt="Picture2" src="https://user-images.githubusercontent.com/61670089/222327510-2a1d31f2-98fe-469e-a3b0-86f0a25d83fd.png">
<img width="868" alt="Screen Shot 2023-03-01 at 10 58 14 PM" src="https://user-images.githubusercontent.com/61670089/222327626-fee6e894-debb-4558-b944-2513423c46fd.png">
</p>
Above is the word cloud plots for topics from the e-cig portion’s topic model manually. The size of the
words is proportional to the frequency of the words in transcripts. Compared with the top topic plot
shown previously, this word cloud not only adds the quantitative variable “frequency” of each word into
the visualization but also offers a close look on each topic with more words from them.
<p float="left">
  <img width="442" alt="Picture3" src="https://user-images.githubusercontent.com/61670089/222328066-307e9bf4-0d67-4768-a0db-a0cae00ca857.png">
<img width="445" alt="Picture4" src="https://user-images.githubusercontent.com/61670089/222328068-a78a7237-9a5c-4c3f-9c0c-8175e3e7dcec.png">
<img width="454" alt="Picture5" src="https://user-images.githubusercontent.com/61670089/222328069-943e72d5-ddb7-4245-82d1-7e53d9ae51e1.png">
<img width="438" alt="Picture6" src="https://user-images.githubusercontent.com/61670089/222328070-2c93ed8a-b309-4bbd-a0bc-0f443d9cea3c.png">
</p>

Further from our model, we tried to estimate the relationship between our metadata and the topics. We
want to find the expected proportion of each topic among different states. We used the estimate effect,
which includes the expected proportion of a hearing that belongs to a topic as a function of covariate
“state”, where topic prevalence for a particular topic is contrasted for groups (different states, we only
have 32 states in total in our dataset). Above are four typical fruitiful expected proportions plots.

From the above four plots, we found that some topics are highly state-related (topic 4 and topic 15), while
others are evenly discussed in most states (topic 7 and topic 10). However, although both topic 4 and topic
15 are highly state-related, they are related to states in different ways. Topic 15 is about e-cig usage and
secondhand cigarette in park. It is mostly discussed in Hawaii since as a traveling state, Hawaii has some
unique concerns about e-cig usage in tourist facilities. On the other hand, topic 4 is highly discussed in
New Jersy state simply because it contains main topic words “New” and “Jersy”, which from content of
words ties it to a single state.

## Experiments and Evaluation

#### Experimenting to find the best structural topic model

With the auto transcriptions in hand, we are able to start building our structural topic models, which will help us further process our data and identify the main topics from those hearings. However, one pivotal parameter we need to determine is “K”, the number of topics we want to extract from the corpus of transcriptions.

In order to find the best number for parameter “K”, we first trained a group of topic models with different numbers of topics, and then evaluate these models. Just similar to the k-means clustering and many other unsupervised models, we are not able to know the “K”, how many topics we should use, ahead of time; and there is no single “correct” answer with a specific K for any given corpus. Therefore, we tried a number of different values of topic numbers
, and enumerative find the “K” that best fits our data. Using parallel processing, we are able to train these seven topic models with different “K” in a reasonable amount of time. These seven models are trained only with part of our data and leave held-out data for us to do the evaluation. As shown below, we are taking the held-out likelihood and the lower bound into consideration. Here we can see that the held-out likelihood is at its highest value when “K” equals 20 and 40, and the lower bound is at its lowest value when “K” equals 20. Therefore, we are able to say that the structural topic
modeling has the best performance over our corpus with parameter “K” being 20.
<p align="center">
  <img width="416" alt="Picture7" src="https://user-images.githubusercontent.com/61670089/222328604-3cd2ea52-6369-458a-b3fa-b3aa96a3fedb.png">
</p>

After we tried the best “K” on whole transcripts, we also need to adjust our “k” when focusing on the
e-cig portion of the transcripts. Since the best “k” we got previously is 20, now we do the diagnose again
but to be more precise, we made the increment to be 1 from 10 to 30. Here, the Semantic Coherence
measures the correlation between words with in each topic; it is maximized when the most probable
words in a given topic frequently co-occur together;
and the Lower Bound is actually the lower bound of
exclusivity. Here we can see that it is a trade-off
between semantic coherence and exclusivity. Since
all the topics are subtopics of e-cig, we have more
tolerance for exclusivity among different topics than
coherence within each topic. Therefore, we finally
choose 15 to be our “K” parameter for e-cig portion
of transcripts after considering the balance between
coherence and exclusivity. (In response to the
comment: Here we added the evaluation for e-cig portion’s model and explained what is the diagnostic value “lower bound”, and how to evaluate the models base on it)
 
<p align="center">
<img width="386" alt="Picture8" src="https://user-images.githubusercontent.com/61670089/222328841-0d093933-021c-4104-aaae-30eb229fde08.png">
</p>

## Discussion and Conclusion

Through practicing the methods and algorithms of this project, all of us learned a lot from the process. Take word error rate as an example, it served as a really good tool for us to evaluate the accuracy of the transcripts we acquired by Google and Youtube APIs through comparing them with the human transcripts. However, the limitation of this method is also obvious. We did not have enough human transcripts as the accurate reference, and we cannot test the accuracy of the automatics transcription of both APIs precisely by using a limited amount of human transcripts. Aside from the word error rate, the structural topic model can be deemed as the other main method we learned a lot from practicing. The advantage of this method is apparent, we are able to employ this method to obtain sets of topics from handling a large scale of transcripts synchronously, and we can add metadata we want in this model for defining the effect of metadata on these topics. We managed to estimate the relationship between the metadata and topics even though there are some limitations on the input format of this model like date can only be counted as integer such as “365” days (since Jan 1st 2007 ) instead of “01-01-2008”.
One thing that ended up being harder than we expected was extracting metadata from the texts. Since we do not have the metadata directly that can be inserted into the model. We had to extract state and date from the title of each video. However, the problem was that the title of each video was not fixed. The date format and state abbreviation format was also varied. We thus need to transform many types of formats of dates and state into uniformed ones, and that cleaning process was much harder than we thought. Even though the process of generating the metadata was hard, we are really surprised that the result of the estimated effect of metadata like state on topics seems really well. We managed to find that some topics
are highly stated-related.

Through employing the tools of this project, all of us also learned a lot from the process. Take the Google API as an example, we find that it has the advantage of transcribing each video into text with punctuations and it has the self-adaptation function for adjusting its parameters for high accuracy of the transcripts, but we need to transfer each video into flac or wav type first and store them into google cloud drive which is a tedious and long process and cost a lot of budget. On the contrary, Youtube API is free and it can easily obscure the transcripts of so many videos at the same time. However, some of the transcripts of the video are not missing and we cannot revise its parameters for improving higher accuracy.

If we were in charge of a research lab, the next step and the directions that we may invest in this problem for making major progress might be to generalize the model and adjust the APIs. It aims to further explore other topics that need to be researched other than E-cig. Once the model is generalized and the APIs are adjusted, this model may be employed to explore topics and trend other projects that are related with huge amounts of text data or transcripts.







