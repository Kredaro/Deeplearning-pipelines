# Deeplearning-pipelines
Series of projects and tutorials around using Building production grade Deep learning pipelines


# Plan
#### Content for blog&nbsp;1

- A brief architecture for a simple/scalable Deep learning pipeline.&nbsp;
- Building blocks of a real world Deep learning pipeline.&nbsp;
- Explain the different building blocks.&nbsp;
- HDFS vs Minio for storage layer.&nbsp;
- Training on batch data, storing Data into Minio.&nbsp;
- Data exploration using Spark SQL, Minio and Jupyter notebook.
- Data preparation.

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
