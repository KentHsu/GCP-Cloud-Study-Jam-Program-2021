# Serverless Firebase Development: Challenge Lab

## Provision the environment
```
gcloud config set project $(gcloud projects list \
    --format='value(PROJECT_ID)' \
    --filter='qwiklabs-gcp')
export PROJECT_ID=$(gcloud config get-value project)
git clone https://github.com/rosera/pet-theory.git
```

## Challenge scenario
In this lab you will create a frontend solution using a Rest API and Firestore database. Cloud Firestore is a NoSQL document database that is part of the Firebase platform where you can store, sync, and query data for your mobile and web apps at scale. Lab content is based on resolving a real world scenario through the use of Google Cloud serverless infrastructure.

You will build the following architecture:

![alt text](https://github.com/KentHsu/GCP-Cloud-Study-Jam-Program-2021/blob/main/Serverless%20Firebase%20Development/architecture.png)


## Task 1: Create a Firestore database
In this scenario you create a Firestore Database in Google Cloud. The high level architecture diagram below summarizes the general architecture.

To complete this section successfully, you are required to implement the following:

* Cloud Firestore Database
* Use Firestore Native Mode
* Add location Nam5 (United States)


### Answer for Task 1

```
Create via console
```


## Task 2: Populate the Database
In this scenario, populate the database using test data.

A high level architecture diagram below summarizes the general architecture.

Populate the Database
Example Firestore schema

Collection | Document | Field
--- | --- | ---
data | 70234439 | [dataset]

Netflix Shows Dataset includes the following information

Field | Description
--- | ---
show_id: | Unique ID for every Movie / Tv Show
type: | Identifier - A Movie or TV Show
title: | Title of the Movie / Tv Show
director: | Director of the Movie
cast: | Actors involved in the movie / show
country: | Country where the movie / show was produced
date_added: | Date it was added on Netflix
release_year: | Actual Release year of the move / show
rating: | TV Rating of the movie / show
duration: | Total Duration - in minutes or number of seasons

To complete this section successfully, you are required to implement the following tasks:

Use the sample code from ```pet-theory/lab06/firebase-import-csv/solution```

```
npm install
```

To import CSV use the node ```pet-theory/lab06/firebase-import-csv/solution/index.js```

```
node index.js netflix_titles_original.csv
```

### Answer for Task 2

```
cd pet-theory/lab06/firebase-import-csv/solution
npm install
node index.js netflix_titles_original.csv
```


## Task 3: Create a REST API
In this scenario, create an example REST API.

A high level architecture diagram below summarizes the general architecture.


#### Cloud Run Development
Field | Value
--- | ---
Container Registry Image | rest-api:0.1
Cloud Run Service | netflix-dataset-service
Permission | --allow-unauthenticated

To complete this section successfully, you are required to implement the following tasks:

* Access ```pet-theory/lab06/firebase-rest-api/solution-01```
* Build and Deploy the code to Google Container Registry
* Deploy the image as a Cloud Run Service
* ```curl -X GET $SERVICE_URL``` should respond with: {"status":"Netflix Dataset! Make a query."}


### Answer for Task 3

```
cd ~/pet-theory/lab06/firebase-rest-api/solution-01
gcloud builds submit --tag gcr.io/$PROJECT_ID/rest-api:0.1
gcloud run deploy netflix-dataset-service \
    --image gcr.io/$PROJECT_ID/rest-api:0.1 \
    --allow-unauthenticated
```


## Task 4: Firestore API access
In this scenario, deploy an updated revision of the code to access the Firestore DB.

A high level architecture diagram below summarizes the general architecture.

#### Deploy Cloud Run revision 0.2
Field | Value
--- | ---
Container Registry Image | rest-api:0.2
Cloud Run Service | netflix-dataset-service
Permission | --allow-unauthenticated

To complete this section successfully, you are required to implement the following tasks:

* Access ```pet-theory/lab06/firebase-rest-api/solution-02```
* Build the updated application
* Use Cloud Build to tag and deploy image revision to Container Registry
* Deploy the new image as Cloud Run service
* ```curl -X GET $SERVICE_URL/2019``` should respond with json dataset


### Answer for Task 4

```
cd ~/pet-theory/lab06/firebase-rest-api/solution-02
gcloud builds submit --tag gcr.io/$PROJECT_ID/rest-api:0.2
gcloud run deploy netflix-dataset-service \
    --image gcr.io/$PROJECT_ID/rest-api:0.2 \
    --allow-unauthenticated
```


## Task 5: Deploy the Staging Frontend
In this scenario, deploy the Staging Frontend.

A high level architecture diagram below summarizes the general architecture.

#### Deploy Frontend
Field | Value
--- | ---
REST\_API_SERVICE | REST API SERVICE URL
Container Registry Image | frontend-staging:0.1
Cloud Run Service | frontend-staging-service

To complete this section successfully, you are required to implement the following tasks:

* Access ```pet-theory/lab06/firebase-frontend```
* Build the frontend staging application
* Use Cloud Build to tag and deploy image revision to Container Registry
* Deploy the new image as Cloud Run service
* Frontend access to Rest API and Firestore Database

Access the Frontend Service URL.

Note: It's using a demo dataset to provide the onscreen entries


### Answer for Task 5

```
cd ~/pet-theory/lab06/firebase-frontend
gcloud builds submit --tag gcr.io/$PROJECT_ID/frontend-staging:0.1
gcloud run deploy frontend-staging-service \
    --image gcr.io/$PROJECT_ID/frontend-staging:0.1
```

## Task 6: Deploy the Production Frontend
In this scenario, update the Staging Frontend to use the Firestore database.

A high level architecture diagram below summarizes the general architecture.

#### Deploy Frontend
Field | Value
--- | ---
REST\_API_SERVICE | REST API SERVICE URL
Container Registry Image | frontend-production:0.1
Cloud Run Service | frontend-production-service

To complete this section successfully, you are required to implement the following tasks:

* Access ```pet-theory/lab06/firebase-frontend/public```
* Update the frontend application i.e. ```app.js``` to use the REST API
* Don't forget to append the year to the SERVICE_URL
* Use Cloud Build to tag and deploy image revision to Container Registry
* Deploy the new image as Cloud Run service
* Frontend access to Rest API and Firestore Database

Now that the services have been deployed you will be able to see the contents of the Firestore database using the frontend service.


### Answer for Task 6

Update URL in public/app.js and then run Cloud Build

##### app.js
```javascript
const REST_API_SERVICE = "https://frontend-staging-service-esrtu26iaq-uc.a.run.app/2020"
```

```
gcloud builds submit --tag gcr.io/$PROJECT_ID/frontend-production:0.1
gcloud run deploy frontend-production-service \
    --image gcr.io/$PROJECT_ID/frontend-production:0.1
```

