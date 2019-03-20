
#### Content for blog&nbsp;1

- A brief architecture for a simple/scalable Deep learning pipeline.&nbsp;
- Explain the different building blocks.&nbsp;
- HDFS vs Minio for storage layer.&nbsp;
- Hello world tensorflow + Minio.&nbsp;
- Docker container to run the example.&nbsp;
- Code example to run Hello World tensorflow on Minio play.&nbsp;
- Push the code into Recipes. &nbsp;
- Read the data from Minio using Tensorflow into memory and run a simple Logistic regression training job.
- Create Google collab notebooks for easy running of code.
- Live coding video with explanation on the same.
- Push the code to repo.

#### Content for second&nbsp;blog

- Saving the trained model in the last blog into Minio.&nbsp;
- Serving from the trained model that is stored in Minio.&nbsp;
- Check pointing the training into Minio.&nbsp;
- Restoring the checkpoints from Minio.&nbsp;
- A Neural network project.&nbsp;
- Save the code examples in Github.&nbsp;
- Create Google collab notebooks for the project.
- Live coding video with explanation on the same.
- Push the code to repo.

#### Content for third&nbsp;blog

- Running the project in last example end to end on Kubeflow.&nbsp;
- Create Terraform templates to recreate the Kubeflow setup with Minio in one shot on GKE.
- Live coding video with explanation on the same.
- Push the code to repo.

#### Content for fourth&nbsp;blog.

- Batch reading of data stored in Minio for training and scaling the in memory training.&nbsp;
- Neural network training on Large dataset.&nbsp;
- Create Containers for the code to run, push to repo.
- Live coding video with explanation on the same.

#### Content for the fifth&nbsp;blog

- Distributing the tensorflow training with large datasets stored in Minio distributed setup.&nbsp;
- Terraform scripts for getting the one shot setup.&nbsp;
- Kubeflow project for the same.
- Live coding video with explanation on the same.
- Push the code to repo.

#### Content for sixth&nbsp;blog

- Ingest real world data from Kafka and store it in Minio as shredded files.&nbsp;
- Run batch Deep learning jobs on the injested stream data into Minio.&nbsp;
- Live coding video with explanation on the same.
- Terraform scripts for the setup.&nbsp;
- Kubeflow project.
- Push the code to repo.

#### Content for seventh&nbsp;blog

- Clean the data using Kafka streaming and Kafka-SQL.&nbsp;
- Store the cleaned data on Minio.&nbsp;
- Run Deep learning training jobs.&nbsp;
- Terraform scripts and live code video.&nbsp;
- Push the code to repo.

#### Content for eighth, Ninth and 10th&nbsp;blog

- Large scale Image processing pipeline using Convolutional Neural networks using Minio and Tensorflow.
- Live coding video.&nbsp;
- Terraform scripts and Kubeflow projects.&nbsp;
- Push the code to repo.

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
