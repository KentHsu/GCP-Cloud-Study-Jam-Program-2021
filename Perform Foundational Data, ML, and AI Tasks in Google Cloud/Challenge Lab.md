# Perform Foundational Data, ML, and AI Tasks in Google Cloud: Challenge Lab

## Challenge scenario
As a junior data engineer in Jooli Inc. and recently trained with Google Cloud and a number of data services you have been asked to demonstrate your newly learned skills. The team has asked you to complete the following tasks.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.


## Task 1: Run a simple Dataflow job
You have used Dataflow in the question to load data into BigQuery from Pub/Sub, now use the Dataflow batch template **Text Files on Cloud Storage to BigQuery** under "Process Data in Bulk (batch)" to transfer data from a Cloud Storage bucket (```gs://cloud-training/gsp323/lab.csv```). The following table has the values you need to correctly configure the Dataflow job.

You will need to make sure you have:

* created a BigQuery dataset called lab
* replace YOUR_PROJECT with the lab project name
* created a Cloud Storage Bucket called YOUR_PROJECT

Field | Value
---   | ---
JavaScript UDF path in Cloud Storage | gs://cloud-training/gsp323/lab.js
JSON path	| gs://cloud-training/gsp323/lab.schema
JavaScript UDF name	| transform
BigQuery output table | YOUR_PROJECT:lab.customers
Cloud Storage input path | gs://cloud-training/gsp323/lab.csv
Temporary BigQuery directory | gs://YOUR_PROJECT/bigquery_temp
Temporary location | gs://YOUR_PROJECT/temp

Wait for the job to finish before trying to check your progress.

### Answer for Task 1

1. Create BigQuery table and GCS bucket
	
	```
	bq mk lab
	bq mk lab.customers
	gsutil mb gs://[ProjectID]
	```
	
2. Follow the instructions on Dataflow doc
	
	[Answer on Dataflow Doc](https://cloud.google.com/dataflow/docs/guides/templates/provided-streaming?hl=zh-tw#gcstexttobigquerystream)
	
	![alt text](https://github.com/KentHsu/GCP-Cloud-Study-Jam-Program-2021/blob/main/Perform%20Foundational%20Data%2C%20ML%2C%20and%20AI%20Tasks%20in%20Google%20Cloud/Task1.jpg)


## Task 2: Run a simple Dataproc job
You have used Dataproc in the quest, now you must run another example Spark job using Dataproc.

Before you run the job, log into one of the cluster nodes and copy the /data.txt file into hdfs (use the command ```hdfs dfs -cp gs://cloud-training/gsp323/data.txt /data.txt```).

Run a Dataproc job using the values below.
	
Field | Value
---   | ---
Region | us-central1
Job type | Spark
Main class or jar | org.apache.spark.examples.SparkPageRank
Jar files | file:///usr/lib/spark/examples/jars/spark-examples.jar
Arguments | /data.txt
Max restarts per hour | 1
	
Wait for the job to finish before trying to check your progress.


### Answer for Task 2

```
gcloud config set dataproc/region us-central1

gcloud dataproc clusters create lab-cluster --worker-boot-disk-size 500

gcloud dataproc jobs submit spark --cluster lab-cluster \
  --region us-central1 \
  --class org.apache.spark.examples.SparkPageRank \
  --jars file:///usr/lib/spark/examples/jars/spark-examples.jar \
  --max-failures-per-hour 1 \
  -- /data.txt
```


## Task 3: Run a simple Dataprep job
You have used Dataprep to import data files and transformed them to gain views of the data. Use Dataprep to import one CSV file (described below) that holds data of lab executions.

```gs://cloud-training/gsp323/runs.csv``` structure:


runid | userid | labid | lab_title | start | end | time | score | state
--- | --- | --- | --- | --- | --- | --- | --- | --- |
5556 | 545 | 122	| Lab 122	| 2020-04-09 16:18:19 | 2020-04-09 17:10:11 | 3112 | 61.25 | SUCCESS
5557 | 116 | 165 | Lab 165 | 2020-04-09 16:44:45 | 2020-04-09 18:13:58 | 5353 | 60.5 | SUCCESS
5558 | 969 | 31 | Lab 31 | 2020-04-09 17:59:01 | 2020-04-09 18:02:09 | 188 | 0 | FAILURE


Perform the following transforms to ensure the data is in the right state:

* Remove all rows with the state of "FAILURE"
	
	```Directly select column 10 ```
	
* Remove all rows with 0 or 0.0 as a score (Use the regex pattern ```/(^0$|^0\.0$)/```)

	```filter_rows -> contains -> regex```
	
* Label columns with the names above
	
	```rename columns -> manually rename```

Make sure you run the job. You will need to wait until the Dataflow job completes before you can grade this task.


## Task 4: AI
Complete one of the tasks below, YOUR_PROJECT must be replaced with your lab project name.

* Use Google Cloud Speech API to analyze the audio file gs://cloud-training/gsp323/task4.flac. Once you have analyzed the file you can upload the resulting analysis to gs://YOUR_PROJECT-marking/task4-gcs.result.

	### Answer
	
	1. Create a API key via console and export it to environment variable
	
		```sh
		export API_KEY=<YOUR_API_KEY>
		```
	2. Create request.json

		```json
		{
		  "config": {
		      "encoding":"FLAC",
		      "languageCode": "en-US"
		  },
		  "audio": {
		      "uri":"gs://cloud-training/gsp323/task4.flac"
		  }
		}
		```
	3. Send request to Google Cloud Speech API
		
		```sh
		curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json \
		"https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > result.json
		```
	4. Upload result to specific Google Cloud Storage

		```
		gsutil cp result.json gs://YOUR_PROJECT-marking/task4-gcs.result
		```

* Use the Cloud Natural Language API to analyze the sentence from text about Odin. The text you need to analyze is "Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." Once you have analyzed the text you can upload the resulting analysis to gs://YOUR_PROJECT-marking/task4-cnl.result.

	### Answer
	
	1. Create API key
		
		```sh
		export GOOGLE_CLOUD_PROJECT=$(gcloud config get-value core/project)
		```
		```
		gcloud iam service-accounts create my-natlang-sa \
		  --display-name "my natural language service account"
  
  		gcloud iam service-accounts keys create ~/key.json \
		  --iam-account my-natlang-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
		```
		```sh
		export GOOGLE_APPLICATION_CREDENTIALS="/home/USER/key.json"
		```
	
	1. Send Request to Cloud Natural Language API
	
		```
		gcloud ml language analyze-syntax --content="Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." > result.json
		```
	
	2. Upload result to specific Google Cloud Storage

		```
		gsutil cp result.json gs://YOUR_PROJECT-marking/task4-cnl.result
		```

* Use Google Video Intelligence and detect all text on the video gs://spls/gsp154/video/train.mp4. Once you have completed the processing of the video, pipe the output into a file and upload to gs://YOUR_PROJECT-marking/task4-gvi.result. Ensure the progress of the operation is complete and the service account you're uploading the output with has the Storage Object Admin role.

	### Answer
	
	1. Set up authentication
		
		```
		gcloud iam service-accounts create quickstart
		
		gcloud iam service-accounts keys create key.json \
		--iam-account quickstart@<ProjectID>.iam.gserviceaccount.com
		
		gcloud auth activate-service-account --key-file key.json
		
		gcloud auth print-access-token
		```
	
	1. Create a request.json
		
		```json
		{
		   "inputUri":"gs://spls/gsp154/video/train.mp4",
		   "features": [
		       "TEXT_DETECTION"
		   ]
		}
		```
		
	2. Send videos:annotate request
		
		```sh
		curl -s -H 'Content-Type: application/json' \
	    -H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
	    'https://videointelligence.googleapis.com/v1/videos:annotate' \
	    -d @request.json
		```

	3. Copy result from previous step into below commend

		```sh
		curl -s -H 'Content-Type: application/json' \
    	-H 'Authorization: Bearer '$(gcloud auth print-access-token)'' \
    	'https://videointelligence.googleapis.com/v1/projects/PROJECTS/locations/		LOCATIONS/operations/OPERATION_NAME' > result.json
		```
				
	4. Upload result to specific Google Cloud Storage

		```
		gsutil cp result.json gs://<ProjectID>-marking/task4-gvi.result
		```
		
