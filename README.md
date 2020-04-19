# Distributed-Image-Processing
In this hands-on lab, you will learn how to use Apache Spark on Cloud Dataproc to distribute a computationally intensive image processing task onto a cluster of machines.

### Introduction

Cloud Dataproc is a managed Spark and Hadoop service that lets you take advantage of open source data tools for batch processing, querying, streaming, and machine learning. Cloud Dataproc automation helps you create clusters quickly, manage them easily, and save money by turning clusters off when you don't need them. With less time and money spent on administration, you can focus on your jobs and your data.

<hr>

<h6>Create a development machine in Compute Engine</h6>

Configure the following fields, leave the others at their default value.
<ol>
  <li>Name: devhost</li>
  <li>Machine Type: 2 vCPUs (n1-standard-2 instance)</li>
  <li>Identity and API Access: Allow full access to all Cloud APIs.</li>
</ol>

<hr>

<h6>Install Software</h6>

Now set up the software to run the job. Using sbt, an open source build tool, you'll build the JAR for the job you'll submit to the Cloud Dataproc cluster. This JAR will contain the program and the required packages necessary to run the job. The job will detect faces in a set of image files stored in a Google Cloud Storage (GCS) bucket, and write out image files with the faces outlined, to either the same or to another Cloud Storage bucket.

<hr>
Step 1: Set up Scala and sbt.  In the SSH window, install Scala and sbt with the following commands so that you can compile the code:
<ol>
  <li>sudo apt-get install -y dirmngr</li>
  <li>sudo apt-get update</li>
  <li>sudo apt-get install -y apt-transport-https</li>
  <li>echo "deb https://dl.bintray.com/sbt/debian /" | \
sudo tee -a /etc/apt/sources.list.d/sbt.list</li>
  <li>sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823</li>
  <li>sudo apt-get update</li>
  <li>sudo apt-get install -y bc scala sbt</li>
</ol>

<hr>

<h6>Step 2: Set up the Feature Detector Files</h6>

Now you'll build the Feature Detector files. The code for this lab is available in the Cloud Dataproc repository on github. You'll clone the repository, then cd into the directory for this lab and build a "fat JAR" of the feature detector so that it can be submitted to Cloud Dataproc. Run the following commands in the SSH window:
<ul>
  <li>sudo apt-get update</li>
  <li>git clone https://github.com/GoogleCloudPlatform/cloud-dataproc</li>
  <li>cd cloud-dataproc/codelabs/opencv-haarcascade</li>
 </ul>

<hr>

<h6>Step 3: Launch build</h6>
<ul>
  <li>sbt assembly</li>
 </ul>


<hr>
  
<h6>Create a GCS bucket and collect images</h6>
<ul>
  <li>Fetch the Project ID to use to name your bucket.</li>
  <li>GCP_PROJECT=$(gcloud config get-value core/project)</li>
  <li>Name your bucket and set a shell variable to your bucket name. The shell variable will be used in commands to refer to your bucket.</li>
  <li>MYBUCKET="${USER//google}-image-${RANDOM}"</li>
    <li>echo MYBUCKET=${MYBUCKET}</li>
</ul>

<h6>Step 2</h6>
<ul>
  <li>Use the gsutil program, which comes with gcloud in the Cloud SDK, to create the bucket to hold your sample images:</li>
  <li>gsutil mb gs://${MYBUCKET}</li>
</ul>

<h6>Step 3</h6>
<ul>
  <li>Download some sample images into your bucket:</li>
  <li>curl https://www.publicdomainpictures.net/pictures/20000/velka/family-of-three-871290963799xUk.jpg | gsutil cp - gs://${MYBUCKET}/imgs/family-of-three.jpg</li>
    <li>curl https://www.publicdomainpictures.net/pictures/10000/velka/african-woman-331287912508yqXc.jpg | gsutil cp - gs://${MYBUCKET}/imgs/african-woman.jpg</li>
  <li>curl https://www.publicdomainpictures.net/pictures/10000/velka/296-1246658839vCW7.jpg | gsutil cp - gs://${MYBUCKET}/imgs/classroom.jpg</li>
</ul>

<h6>Step 4</h6>
<ul>
  <li>Run this to see the contents of your bucket:</li>
  <li>gsutil ls -R gs://${MYBUCKET}</li>
</ul>
