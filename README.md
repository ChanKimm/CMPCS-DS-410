This is the group project I completed with my group.

---------------------------------------------------------
Project title : Prediction Model for Delayed Flights

Course: DS/CMPSC 410

Semester: Fall 2025

Team Members: Youhyun Kim, Atharv Madne, William Van Eck, Chanhyung Kim, Ga Ryoung Lee, Patrick Mariak

Team Name: Data Pilots

1. Introduction and Motivation

Our work is situated in the field of aviation data analytics. We focus on predicting flight delays during peak travel periods, especially around Christmas, when airports experience heavy traffic, unstable weather, and limited capacity. These conditions often lead to increased disruptions at regional airports such as state college and nearby hubs.

Provide background on the application domain (e.g., social media analytics, climate data, genomics, finance, IoT).
This project lies within the domain of aviation analytics, which focuses on extracting insights from large scale flight and airport data. The aviation industry generates an extremely large amount of historical records on flight schedules, weather conditions, passenger demand, and operational performance. These datasets enable researchers to study patterns such as delay causes and airport capacity challenges.

The key problem/question our project aims to address is:
Can we estimate the likelihood of a flight being delayed during Christmas travel period at State College Airport and nearby major hubs?

This problem benefits from big data approaches because the flight dataset spans multiple years and contains millions of records with many variables such as airport, airline, departure time, weather, and delay duration. Traditional tools cannot efficiently load, filter, or model data at the scale due to memory limitations and long processing times.

2. Project Objectives

Build PySpark pipeline for cleaning, transformation, and feature creation.
Gather and preprocess large flight datasets.
Train scalable ML models to predict delay risk.
Visualize insights and evaluate performance.


3. Data Description
Data Source: Bureau of Transportation Services
Data Size: Approximately 14.5 million records, between 2015-2024 excluding 2020 and 2021
Data Characteristics: Structured, temporal data based on individual flights around the State College area. Flight date, Carrier, Scheduled arrival time, scheduled departure time, actual arrival time, actual departure time, cancellation/diverted indicators, delay times, flight distance, and air time are all features in the data.
Data Challenges: Skewed towards flights that were not delayed, difficulty reading the excel file, some dates had incomplete records, occasionally there were timeouts for downloading the data. Limited to within the previous 10 years to maintain consistent input of information as well as consistent rules for flights. 

4. Technical Approach
Tools & Frameworks: PySpark (RDDs, DataFrames, MLlib, GraphX if relevant), possibly integration with external libraries (e.g., Pandas, Matplotlib, scikit-learn for evaluation).
Primary Framework: Python (Single-node execution).
Key Libraries: Pandas, XGBoost (eXtreme Gradient Boosting) , Scikit-Learn, NumPy


Cluster Usage Plan:

Number of nodes (up to 4 CPU nodes).
	Up to 4 nodes
Expected job types (batch processing, iterative ML training, streaming if applicable).
The pipeline loads 15 partitioned CSV files(Filter_dataset 1 to 15) sequentially, combines them into memory, and performs a one-time training and evaluation batch.
Storage requirements and file formats (e.g., Parquet, CSV, JSON).
Input : CSV, Output : CSV (Confusion Matrix)
Pipeline Overview:
Data ingestion and preprocessing.
Ingestion: We used an efficient, memory-optimized loading strategy to ingest 15 partitioned CSV files (approx. 14.5 million rows) using Pandas. We restricted memory usage by loading only the 11 essential columns
Preprocessing: We filtered out cancelled and diverted flights to focus on operational delays.
Feature Engineering: We engineered temporal features (converting CRS_DEP_TIME to integer hours) and created a custom binary feature, winter_holiday, to capture seasonal traffic spikes.

Exploratory analysis (PySpark SQL, summary statistics).
Summary Statistics: We analyzed the target variable distribution, identifying a significant class imbalance where only about 20% of flights were delayed (more than 15 minutes), and 80% were on time.
Data Validation: We ensured no null values exist in the target column (DEP_DELAY) before training.

Model building or large-scale computation.
Algorithm: We used the Random Forest Regression/Classification models, as well as XGBoost Classifier (Extreme Gradient Boosting), a scalable tree-boosting algorithm.
Computation: The model was trained on an 80% split (approx. 11.4 million rows) using 200 estimators and a tree depth of 7.
Imbalance Handling: To address the minority class issue, we computed and applied a scale_pos_weight of 2.5, forcing the model to prioritize delay detection.

Visualization and interpretation of results.
Model Comparison: We used accuracy, f1-score, precision, and recall to compare the Random Forest and XGBoost. 
Feature Importance: We extracted the feature importance scores from the trained model, determining that origin airports, airlines, and months are primary drivers of the prediction. Although winter holiday was not a top driver of flight delays, we still analyzed its impact for our original research question. 
Actual interpretations of results can be found below. 

5. Results and discussion

1.What did your data visualization reveal?

Our results reveal that Random Forest outperforms XGBoost. Although Random Forest has lower precision than XGBoost, we would still recommend future research to consider Random Forest as its prediction model. 

Some of the features highlighted by the two models as important were: origin airports, airlines, and months. To answer our original research questions of whether winter holidays affect delays, we also dived into that variable as well. 

From the graph above, we can conclude that there is a minor difference in flight delays between flights in the winter holiday season (December 1 - January 10) and outside that time frame. Our models also supported this finding by ranking winter holiday at the bottom of the feature importance list. 

      
One of the main predictors of flight delays were operating carriers. Certain airlines result in higher delay rates compared to others. We found the top and bottom 5 airlines on delay rates in the table below. We realized that airlines with high delay rates are typically from low-cost carriers as expected. Despite numerous negative reviews, Delta Airline had the lowest delay rates
       
Another variable of interest was the month the flight departed. There are two peaks in the graph: one near July and one near December. This further proves that the winter holiday season is not the best indicator of delay. Instead, we see an increased trend in delay rates between May to August. This can be primarily due to the sheer amount of students, workers, and families that leave for summer vacation. 

Our last variable of interest is the origin airport where the flight departed. We are only interested in the airports near State College (EWR, JFK, PHL, SCE, PIT). The graph suggests that big airports in the New York region have higher delay rates compared to smaller airports like the ones in State College and Pittsburgh, PA. This further proves that selecting certain airports can minimize the risk of the flights getting delayed. 

2.What did you learn about your data using your model? How did the model perform?

We learned that strict feature selection was critical to avoid data leakage and efficiently manage RAM across our 14.5 million records. Our model(XGBoost) catches 61% of all delayed flights, successfully flagging most problems before they happen. The cost of being this sensitive is a lower Precision (31%), meaning we generate more false alarms. Overall, our 65% accuracy shows we prioritized safety. 

Both random forest models both performed below expectations. The regression model RMSE values (~50 min), shows the model often predicted almost an hour more of actual delay time. The 3 features the model found most important were month, destination, and airline carrier.
The data imbalance also had a huge impact on the classification model. As stated before, most of the flights were not delayed, ~80% of our dataset. This led the classifier to pick the proportional larger class, in this case non-delays.

3.How did your workflow scale on multiple nodes/cpus of Roar?

For the XGBoost model, we utilized 4 nodes on the Roar-Collab cluster to handle the computational workload. Our primary strategy focused on efficiency: by removing unnecessary columns before training, we can reduce the amount of RAM required.  This allowed us to train the XGBoost model using standard parallel processing on the CPUs, eliminating the need for Cluster Mode. 
In terms of the random forest models, using multiple nodes/cpus scaled the workflow to an extent. Due to the number of trees in each model, sometimes I would run into Java heap issues. This would bring the workflow to a grinding halt, and cause me to rerun the program. However, in terms of processing CSVs, indexing, and assembling the pipeline, Roar made the workflow so much easier.

4. What is the path to your models/code/data on Github?
https://github.com/grlee1128/CMPCS-DS-410.git

5.What unsolved problems remain? How would you revisit this course project to address that?

The models were accurate but not as accurate as we preferred then. The data imbalance was the biggest issue due to which random forest struggled in precision and XGboost while trying to make up for this imbalance struggled with overall accuracy. Revisiting this problem would require more data and more time for parameters tuning to be done over and over to find the razor thin margins that result in better accuracy for such an unpredictable issue.


6. CREDIT Statement

Youhyun Kim : Conceptualization, Methodology, Software, Writing - Original Draft, Review & Editing, Supervision

Patrick Mariak : Conceptualization, Methodology, Software, Validation,  Writing - Original Draft, Review & Editing, Supervision

Atharv Madne : Conceptualization, Methodology, Writing - Review & Editing, Project administration 

William Van Eck: Conceptualization, Data Acquisition

Chanhyung Kim:  Conceptualization, Writing - Review & Editing, Data Acquisition & Preprocessing

Ga Ryoung Lee: Conceptualization, Methodology, Writing - Original Draft, Visualization





7. References

Dataset: U.S. Department of Transportation, Bureau of Transportation Statistics, Airline On-Time Performance Data. Dataset with flight Date and delay  https://transtats.bts.gov/DL_SelectFields.aspx?gnoyr_VQ=FGJ&QO_fu146_anzr=b0-gvzr 
Inspiration: https://www.youtube.com/watch?v=EpzzTNhKXAg
Data for Validation set: https://www.transtats.bts.gov/holidayDelay.asp?20=E
