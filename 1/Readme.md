
Doing Deep learning on large data sets at scale has it’s own challenges, But here are the major parts of the Deep learning life cycle,

#### Data ingestion

 ![](https://cdn-images-1.medium.com/max/1600/1*S7lMbuLI2gJYlEWWylj4bQ.jpeg)
*\< note: I need to use correct icons and make a GIF of data moving through the&nbsp;system\>*

#### Preprocessing

The real world data is usually not consistent. The data model could evolve over a period of time, there might be anomalies in the data, missing values and many more issues with it. We need a way to preprocess the stored data. This validation, pre-processing and cleaning of data can be best done at the Kafka streaming layer rather than doing it once the data is persisted to application database. Kafka Streaming, Kafka-SQL (KSQL) makes it convenient to achieve the data preprocessing on the data and store the results back into a new Kafka topic.

 ![](https://cdn-images-1.medium.com/max/1600/1*THNM6qmcvaAV9IK2M4tVjQ.jpeg)

#### Data Storage

In real world applications here are the popular sources of data where application data reside,

- Application databases stores the app data.
- Object Storage with image data.
- Kafka topics with the true source of data.

But here are the expectations from the storage system which would feed the data into Deep learning training/processing systems,&nbsp;

\<Illustrations for each of the below point\>&nbsp;

- Should be able to ingest, store/archive large amounts of historical data. &nbsp;
- Deep learning processing systems should be able to bulk read files/blobs containing the training data in batches during training.&nbsp;
- Deep learning processing systems should be able to checkpoint various stages of Deep learning into the system so that the training can be restarted from a given point.&nbsp;
- Should be able to store and restore the trained model.&nbsp;

The only two systems that fit into serving the above objectives&nbsp;

#### Deep learning&nbsp;training

- Serving of trained model

Here are the two crucial aspects of doing Deep learning at scale.,

- Storage
- Computing
