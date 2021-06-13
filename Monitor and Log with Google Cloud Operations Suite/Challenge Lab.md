# Monitor and Log with Google Cloud Operations Suite: Challenge Lab

## Challenge scenario
In your new role as Junior Cloud Engineer for Jooli Inc., you're expected to help manage the Cloud infrastructure components and support the video operations team. Common tasks include monitoring resource utilization, analyzing logs, configuring alerts, and reporting on any issues related to Jooli Inc.'s online services.

As you're expected to have the skills and knowledge for these tasks, step-by-step guides are not provided.

Some Jooli Inc. standards you should follow:

* Create all resources in the us-east1 region and us-east1-b zone, unless otherwise directed.
* Naming is team-resource, e.g. an instance could be named video-webserver1.
* Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. Unless directed, use n1-standard-1.


## Your challenge
On the first day of your new job, your manager gives you a series of tasks that you must complete. Good luck!

Your primary concern is a media upload function that Jooli Inc. provides. This function allows subscribers to upload video content to edit and transform using Jooli Inc.'s innovative range of cloud based media production tools.

The media upload function is a critical part of the service, and it is vital that Jooli Inc. is aware of any changes in the behavior of the users that might impact performance or cost of the services.

Your tasks today will use Cloud Operations tools to improve the company's ability to identify such changes and respond to them rapidly. Your manager has told you that the company is concerned that recent changes in end user behavior, combined with a new generation of phones and tablets, is fuelling a demand for much higher media such as 4K, and even 8K, video. Storage for the data is a relatively minor concern but the company wants to make sure that resource consumption by the Cloud Functions used for media upload and transcoding do not run into any limits or result in unexpected spikes in billing costs.


## Task 1: Configure Cloud Monitoring
Your first task is to enable Cloud Monitoring for your project.

A basic Cloud Monitoring dashboard, called **Media_Dashboard**, will be made available to you automatically, but you have to enable Cloud Monitoring in your project before you will be able to access this dashboard.

Once you initialize Cloud Monitoring, you can access the initial dashboard, called **Media_Dashboard**. In subsequent tasks you will add custom metrics to this basic dashboard. The initial dashboard configuration incldues some charts that display stats about the latency of the video upload Cloud Function.

### Answer for Task 1

```
Open Cloud Monitor
```


## Task 2: Configure a Compute Instance to generate Custom Cloud Monitoring metrics
Your next task is to confirm that the monitoring service that checks the length of the video processing queue is working correctly.

The monitoring service creates a custom metric, ```opencensus/my.videoservice.org/measure/input_queue_size```, that allows you to monitor the state of the Jooli Inc.'s video processing queue. This custom metric is created and written to by a Go application that runs on a Compute Instance called **video-queue-monitor**.

The **video-queue-monitor** Compute Instance has been deployed for you and uses a startup script to install and launch the input queue monitoring Go application. This application was tested fully in a development environment but the configuration in your Compute Instance has not been finalized. The Go application will not write custom metric data until the application is correctly configured by the startup script.

You must modify the startup script for the **video-queue-monitor** Compute Instance so that the queue monitoring application (the Go application) can create and write to custom metrics. Once you have updated the startup script you will need to restart the instance.

The Go application is installed in the ```/work/go``` directory in the Compute Instance by the startup script.

You can confirm that the application is working by searching for the metric ```input_queue_size``` in the Metrics Explorer in Cloud Monitoring.

### Answer for Task 2


```
Modify below startup.sh script in video-queue-monitor instance

	export MY_PROJECT_ID=[REPLACE-WITH-PROJECT_ID]
	export MY_GCE_INSTANCE_ID=[REPLACE-WITH-INSTANCE-ID]
	export MY_GCE_INSTANCE_ZONE=[REPLACE-WITH-INSTANCE-ZONE]
```


## Task 3: Create a custom metric using Cloud Operations logging events
Examine the Cloud Operations logs and create a custom metric that tracks the total volume of uploaded media files to your Cloud Function. The video upload Cloud Function creates a Cloud Operations Logging event that includes metadata about the type of video file the video processing system handles. You have been asked to configure a custom log based metric called ```large_video_upload_rate``` that will monitor the rate at which high resolution video files, those recorded at either 4K or 8K resolution, are uploaded. The Cloud Function is already processing this data, and if you search the Cloud Operations logs using the advanced filter mode you will find log entries that contain the string ```"file_format: 4K"``` or ```"file_format: 8K"``` in the ```textPayload``` field whenever the ```video_processing``` Cloud Function receives a request to process a high resolution video. You can use that filter to create your custom metric.

### Answer for Task 3

```
Put textPayload=~"file_format\: ([4,8]K).*" in Query Editor and then Run Query
Click actions -> create metrics with name: large_video_upload_rate
```


## Task 4: Add custom metrics to the Media Dashboard in Cloud Operations Monitoring
You must now add two charts to the Media Dashboard:

1. Add a chart for the video input queue length custom metric that is generated by the Go application running on the **video-queue-monitor** Compute Instance.
2. Add a chart for the high resolution video upload rate custom log based metric to the **Media_Dashboard** custom dashboard.

### Answer for Task 4

```
Add chars in Media Dashboard with 
Resource: VM instances, Metrics: input_queue_size
Resource: VM instances, Metrics: large_video_upload_rate
```


## Task 5: Create a Cloud Operations alert based on the rate of high resolution video file uploads

Create a custom alert using the high resolution video upload metric that triggers when the upload rate for large videos exceeds a count of 3 per minute.

### Answer for Task 5

```
Create policy in Alerting with condition: large_video_upload_rate
```


## Tips and Tricks
* Tip 1. The startup script for the Compute Instance is in the Compute Instance metadata key called ```startup_script```.

* Tip 2. The Compute Instance must have the Cloud Monitoring agent installed and the Go application requires environment variables to be configured with the Google Cloud project, the instance ID, and the compute engine zone.

* Tip 3. The Video Queue length monitoring Go application writes the queue length metric data to a metric called ```custom.googleapis.com/opencensus/my.videoservice.org/measure/input_queue_size``` associated with the ```gce_instance``` resource type.

* Tip 4. To create the custom log based metric, the easiest filter to use is the advanced filter query ```textPayload=~"file_format\: ([4,8]K).*"```. That is a regular expression that matches all Cloud Operations events for the two high resolution video formats you are interested in. You can use the same regular expression and configure labels in the metric definition, which creates a separate time series for each of the two high resolution formats.

* Tip 5. You must use the name provided for the custom log based metric that monitors the rate at which high resolution videos are processed: ```large_video_upload_rate```.


