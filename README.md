# Distributed-Image-Processing
In this hands-on lab, you will learn how to use Apache Spark on Cloud Dataproc to distribute a computationally intensive image processing task onto a cluster of machines.

### Introduction

Cloud Dataproc is a managed Spark and Hadoop service that lets you take advantage of open source data tools for batch processing, querying, streaming, and machine learning. Cloud Dataproc automation helps you create clusters quickly, manage them easily, and save money by turning clusters off when you don't need them. With less time and money spent on administration, you can focus on your jobs and your data.

<hr>

### Create a development machine in Compute Engine

Configure the following fields, leave the others at their default value.
<ol>
  <li>Name: devhost</li>
  <li>Machine Type: 2 vCPUs (n1-standard-2 instance)</li>
  <li>Identity and API Access: Allow full access to all Cloud APIs.</li>
</ol>

<hr>

### Install Software

Now set up the software to run the job. Using sbt, an open source build tool, you'll build the JAR for the job you'll submit to the Cloud Dataproc cluster. This JAR will contain the program and the required packages necessary to run the job. The job will detect faces in a set of image files stored in a Google Cloud Storage (GCS) bucket, and write out image files with the faces outlined, to either the same or to another Cloud Storage bucket.

<hr>

<h6>Step 1</hs> 
Set up Scala and sbt.  In the SSH window, install Scala and sbt with the following commands so that you can compile the code:
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

<h6>Step 2</hs> 
Set up the Feature Detector Files
Now you'll build the Feature Detector files. The code for this lab is available in the Cloud Dataproc repository on github. You'll clone the repository, then cd into the directory for this lab and build a "fat JAR" of the feature detector so that it can be submitted to Cloud Dataproc. Run the following commands in the SSH window:
<ul>
  <li>sudo apt-get update</li>
  <li>git clone https://github.com/GoogleCloudPlatform/cloud-dataproc</li>
  <li>cd cloud-dataproc/codelabs/opencv-haarcascade</li>
 </ul>

<hr>
<h6>Step 3</hs>
Launch build
<ul>
  <li>sbt assembly</li>
 </ul>


<hr>
  
### Create a GCS bucket and collect images
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

###Create a Cloud Dataproc Cluster
<h6>Step 1</hs>
Run the following commands in the SSH window to name your cluster and to set the MYCLUSTER variable. You'll be using the variable in commands to refer to your cluster:
<ul>
  <li>MYCLUSTER="${USER/_/-}-qwiklab"</li>
  <li>echo MYCLUSTER=${MYCLUSTER}</li>
</ul>

<h6>Step 2</hs>
Set a global GCE region to use and create a new cluster:
<ul>
  <li>gcloud config set dataproc/region global</li>
  <li>gcloud dataproc clusters create ${MYCLUSTER} --bucket=${MYBUCKET} --worker-machine-type=n1-standard-2 --master-machine-type=n1-standard-2   </li>
</ul>
If prompted to use a zone instead of a region, enter Y.

This might take a couple minutes. The default cluster settings, which include two worker nodes, should be sufficient for this lab. n1-standard-2 is specified as both the worker and master machine type to reduce the overall number of cores used by the cluster.

### Submit your job to Cloud Dataproc
In this lab the program you're running is used as a face detector, so the inputted haar classifier must describe a face. A haar classifier is an XML file that is used to describe features that the program will detect. You will download the haar classifier file and include its GCS path in the first argument when you submit your job to your Cloud Dataproc cluster.

<h6>Step 1</hs>
Run the following command in the SSH window to load the face detection configuration file into your bucket:
<ul>
  <li>curl https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml | gsutil cp - gs://${MYBUCKET}/haarcascade_frontalface_default.xml</li>
</ul>

<h6>Step 2</hs>
Use the set of images you uploaded into the imgs directory in your GCS bucket as input to your Feature Detector. You must include the path to that directory as the second argument of your job-submission command.

Submit your job to Cloud Dataproc:
<ul>
  <li>cd ~/cloud-dataproc/codelabs/opencv-haarcascade</li>
   <li>gcloud dataproc jobs submit spark \
--cluster ${MYCLUSTER} \
--jar target/scala-2.10/feature_detector-assembly-1.0.jar -- \
gs://${MYBUCKET}/haarcascade_frontalface_default.xml \
gs://${MYBUCKET}/imgs/ \
gs://${MYBUCKET}/out/</li>
</ul>

<h6>Step 3</hs>
Monitor the job, in the Console go to Navigation menu > Dataproc > Jobs.


