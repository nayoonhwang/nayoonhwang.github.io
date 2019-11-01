---
layout: post
title:  "Dataproc"
date:   2019-10-15 11:22:22 +0900
categories: cloud network
---
https://google.qwiklabs.com/focuses/672?parent=catalog

ntroduction
Cloud Dataproc is a managed Spark and Hadoop service that lets you take advantage of open source data tools for batch processing, querying, streaming, and machine learning. Cloud Dataproc automation helps you create clusters quickly, manage them easily, and save money by turning clusters off when you don't need them. With less time and money spent on administration, you can focus on your jobs and your data.

Consider using Cloud Dataproc to scale out compute-intensive jobs that meet these characteristics:

The job is __embarrassingly parallel__—in other words, you can process different subsets of the data on different machines.
You already have Apache Spark code that does the computation or you are familiar with Apache Spark.
The distribution of the work is pretty uniform across your data subsets.
If different subsets require different amounts of processing (or if you don't already know Apache Spark), Apache Beam on Cloud Dataflow is a compelling alternative because it provides autoscaling data pipelines.

In this lab, the job that you will run outlines the faces in the image using a set of image processing rules specified in OpenCV. The Vision API is a better way to do this, since these sort of hand-coded rules don't work all that well, but this lab is an example of doing a compute-intensive job in a distributed way.

Setup

Install Software
Now we'll set up the software to run the job. Using sbt, an open source build tool, you'll build the JAR for the job you'll submit to the Cloud Dataproc cluster. This JAR will contain the program and the required packages necessary to run the job. The job will detect faces in a set of image files stored in a Google Cloud Storage (GCS) bucket, and write out image files with the faces outlined, to either the same or to another Cloud Storage bucket.

Create a Cloud Dataproc cluster
Step 1
Run the following commands in the SSH window to name your cluster and to set the MYCLUSTER variable. You'll be using the variable in commands to refer to your cluster:

MYCLUSTER="${USER/_/-}-qwiklab"
echo MYCLUSTER=${MYCLUSTER}
Step 2
Set a default GCE zone to use (preferably the one you used for your development machine) and create a new cluster:

gcloud config set compute/zone us-central1-a
gcloud dataproc clusters create ${MYCLUSTER} --worker-machine-type=n1-standard-2 --master-machine-type=n1-standard-2
This might take a couple minutes. The default cluster settings, which include two worker nodes, should be sufficient for this lab. n1-standard-2 is specified as both the worker and master machine type to reduce the overall number of cores used by the cluster.

See the Cloud SDK gcloud dataproc clusters create command for information on using command line flags to customize cluster settings.
Submit your job to Cloud Dataproc
In this lab the program you're running is used as a face detector, so the inputted haar classifier must describe a face. A haar classifier is an XML file that is used to describe features that the program will detect. You will download the haar classifier file and include its GCS path in the first argument when you submit your job to your Cloud Dataproc cluster.

Step 1
Run the following command in the SSH window to load the face detection configuration file into your bucket:

curl https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml | gsutil cp - gs://${MYBUCKET}/haarcascade_frontalface_default.xml
Step 2
Use the set of images you uploaded into the imgs directory in your GCS bucket as input to your Feature Detector. You must include the path to that directory as the second argument of your job-submission command.

Submit your job to Cloud Dataproc:

cd ~/cloud-dataproc/codelabs/opencv-haarcascade
gcloud dataproc jobs submit spark \
--cluster ${MYCLUSTER} \
--jar target/scala-2.10/feature_detector-assembly-1.0.jar -- \
gs://${MYBUCKET}/haarcascade_frontalface_default.xml \
gs://${MYBUCKET}/imgs/ \
gs://${MYBUCKET}/out/
You can add any other images to use to the GCS bucket specified in the second argument.

Step 3
Monitor the job, in the Console go to Navigation menu > Dataproc > Jobs. Move on to the next step when you get a similar output:

job-complete.png

Step 4
When the job is complete, go to Navigation menu > Storage and find the bucket you created (it will have your username followed by student-image followed by a random number) and click on it. Then click on an image in the Out directory. The image will download to your computer.

How accurate is the face detection? The Vision API is a better way to do this, since this sort of hand-coded rules don't work all that well. You can see how it works next.

Step 5 (Optional)
Go back to the imgs folder and click on the other images you uploaded to your bucket. This will download the three sample images. Save them to your computer.

Navigate to the Vision API page, scroll down to the Try the API section and upload the images you downloaded from your bucket. You'll see the results of the image detection in seconds. The underlying machine learning models keep improving, so your results may not be the same:

result1.png result2.png result3.png

Step 6 (Optional)
If you want to experiment with improving the Feature Detector, you can make edits to the FeatureDetector code, then rerun sbt assembly and the gcloud dataproc and jobs submit commands.

Congratulations!
You learned how to spin up a Cloud Dataproc cluster and run jobs!

Data_Science_125_2.png

Finish Your Quest
This self-paced lab is part of the Qwiklabs Quest Scientific Data Processing. A Quest is a series of related labs that form a learning path. Completing this Quest earns you the badge above, to recognize your achievement. You can make your badge (or badges) public and link to them in your online resume or social media account. Enroll in this Quest and get immediate completion credit if you've taken this lab. See other available Qwiklabs Quests.

Take Your Next Lab
Continue your Quest with Distributed Computation of NDVI from Landsat Images using Cloud Dataflow, or try one of these:
