# Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

## Challenge scenario
You are just starting your junior cloud engineer role with Jooli inc. So far you have been helping teams create and manage Google Cloud resources.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.

### Your challenge
You are now asked to help a newly formed development team with some of their initial work on a new project around storing and organizing photographs, called memories. You have been asked to assist the memories team with initial configuration for their application development environment; you receive the following request to complete the following tasks:

* Create a bucket for storing the photographs.
* Create a Pub/Sub topic that will be used by a Cloud Function you create.
* Create a Cloud Function.
* Remove the previous cloud engineer’s access from the memories project.

Some Jooli Inc. standards you should follow:

* Create all resources in the us-east1 region and us-east1-b zone, unless otherwise directed.
* Use the project VPCs.
* Naming is normally team-resource, e.g. an instance could be named kraken-webserver1
* Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use f1-micro for small Linux VMs and n1-standard-1 for Windows or other applications such as Kubernetes nodes.
Each task is described in detail below, good luck!

### Get zone and region

```
gcloud config get-value compute/zone
gcloud config get-value compute/region
gcloud compute zones list

```

### Set zone and region

```
gcloud config set compute/region us-east1
gcloud config set compute/zone us-east1-b

```


## Task 1: Create a bucket

You need to create a bucket for the storage of the photographs.

```
gsutil mb -p [PROJECT_ID] gs://memories-photo-bucket
```


## Task 2: Create a Pub/Sub topic

Create a Pub/Sub topic for the Cloud Function to send messages.

```
gcloud pubsub topics create memories-topic
```


## Task 3: Create the thumbnail Cloud Function
Create a Cloud Function that executes every time an object is created in the bucket you created in task 1. The function is written in Node.js 10. Make sure you set the Entry point (Function to execute) to thumbnail and Trigger to Cloud Storage.

In line 15 of index.js replace the text REPLACE\_WITH\_YOUR_TOPIC ID with the Topic ID you created in task 2.


1. Your line 15 will look something like: const topicName = "MyTopic";
2. You must upload one JPG or PNG image into the bucket, we will verify the thumbnail was created (after creating the function successfully). Use any JPG or PNG image, or use this image https://storage.googleapis.com/cloud-training/gsp315/map.jpg; download the image to your machine and then upload that file to your bucket. You will see a thumbnail image appear shortly afterwards (use REFRESH in the bucket details).

```My Solution
gcloud functions deploy memories-function-2 \
	--entry-point thumbnail \
	--trigger-bucket gs://memories-photo-bucket \
	--runtime nodejs10
```
```
wget https://storage.googleapis.com/cloud-training/gsp315/map.jpg
gsutil cp map.jpg gs://memories-photo-bucket
```


#### index.js:

```javascript
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const gcs = require("@google-cloud/storage")();
const PubSub = require("@google-cloud/pubsub");
const imagemagick = require("imagemagick-stream");

exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "memories-topic";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });

          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```

#### package.json:

```json
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/storage": "1.5.1",
    "@google-cloud/pubsub": "^0.18.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```

## Task 4: Remove the previous cloud engineer
You will see that there are two users, one is your account (with the role of Owner) and the other is the previous cloud engineer (with the role of Viewer). We like to keep our security tight, so please remove the previous cloud engineer’s access to the project.