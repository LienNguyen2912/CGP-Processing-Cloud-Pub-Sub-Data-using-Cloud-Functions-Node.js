# Google Cloud Platform - App Dev - Processing Cloud Pub/Sub Data using Cloud Functions: Node.js
We will
- Create a Cloud Function that responds to Cloud Pub/Sub messages.
- Deploy multiple files to a Cloud Function.

![0](https://user-images.githubusercontent.com/73010204/143729587-03a15dfd-d76d-4faa-b140-cacb9f5d153b.png)

## Activate Google Cloud Shell
Google Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Google Cloud Shell provides command-line access to your GCP resources.</br>
In GCP console, on the top right toolbar, click the Open Cloud Shell button.</br>
![00](https://user-images.githubusercontent.com/73010204/143729660-cc3e6f89-a497-4f59-b74b-8d4cabd671dc.png)</br>
Click **Continue**</br>
It takes a few moments to provision and connect to the environment. When you are connected, you are already authenticated, and the project is set to your PROJECT_ID. </br>
**gcloud** is the command-line tool for Google Cloud Platform. It comes pre-installed on Cloud Shell and supports tab-completion.</br>
You can list the active account name with this command:
```sh
gcloud auth list
```
You can list the project ID with this command:
```sh
gcloud config list project
```
Output:</br>
![1](https://user-images.githubusercontent.com/73010204/143729717-75a7d049-4182-43e3-864b-f9eb51527548.png)</br>
## Preparing the case study application
In this section, you access Cloud Shell, clone the git repository containing the Quiz application, configure environment variables, and run the application.</br>
### Clone source code in Cloud Shell
To clone the repository for the class, enter the following command:
```sh
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```
To clone the class repository, execute the following command:
```sh
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```
Create a soft link as a shortcut to the working directory.
```sh
ln -s ~/training-data-analyst/courses/developingapps/v1.2/nodejs/cloudfunctions ~/cloudfunctions
```
### Configure and run the case study application
To change the working directory, enter the following command:
```sh
cd ~/cloudfunctions/start
```
To configure the Quiz application, enter the following command:
```sh
. prepare_environment.sh
```
This script file:
- Creates an App Engine application.
- Exports the environment variables GCLOUD_PROJECT and GCLOUD_BUCKET.
- Runs npm install.
- Creates entities in Cloud Datastore.
- Creates a Cloud Pub/Sub topic.
- Creates a Cloud Spanner Instance, Database, and Table.
- Prints out the Google Cloud Platform Project ID.

Here it is
```sh
echo "Creating App Engine app"
gcloud app create --region "us-central"

echo "Making bucket: gs://$DEVSHELL_PROJECT_ID-media"
gsutil mb gs://$DEVSHELL_PROJECT_ID-media

echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media

echo "Installing dependencies"
npm install -g npm@6.11.3
npm update

echo "Creating Datastore entities"
node setup/add_entities.js

echo "Creating Cloud Pub/Sub topic"
gcloud pubsub topics create feedback

echo "Creating Cloud Spanner Instance, Database, and Table"
gcloud spanner instances create quiz-instance --config=regional-us-central1 --description="Quiz instance" --nodes=1
gcloud spanner databases create quiz-database --instance quiz-instance --ddl "CREATE TABLE Feedback ( feedbackId STRING(100) NOT NULL, email STRING(100), quiz STRING(20), feedback STRING(MAX), rating INT64, score FLOAT64, timestamp INT64 ) PRIMARY KEY (feedbackId);"

echo "Project ID: $DEVSHELL_PROJECT_ID"
```
To run the web application, enter the following command:
```sh
npm start
```

## Working with Cloud Functions
In this section, you create a Cloud Function in your Google Cloud Platform project that is triggered by publishing a message to Cloud Pub/Sub, runs the application, and monitors the Cloud Function invocation.
### Create a Cloud Function
You learned how to provision a Google Compute Engine virtual machine and install software libraries for Node.js, Java, Python software development on Google Cloud Platform.
- Return to the **Cloud Platform Console**.
- On the Navigation menu, select **Cloud Functions**.
- If necessary, click **Enable API**.
- Click **Create function**.
    - **Function name** : _process-feedback_
    - **Trigger type**:	_Cloud Pub/Sub_
    - **Topic**: Select _projects/[PROJECT_ID]/topics/feedback_
![a](https://user-images.githubusercontent.com/73010204/143730149-307b0d26-69f4-4cbf-8bec-04c88d896bc1.png)</br>
- Click Save, and then click **Next**.
- Review the provided function implementation, and then click **Deploy**.
![b](https://user-images.githubusercontent.com/73010204/143730281-a079f462-0696-4a86-b8b4-e89561689f97.png)</br>
![c](https://user-images.githubusercontent.com/73010204/143730299-88f5fab8-d247-4e71-a80d-d2ee35dfca88.png)
### Run the web application
- Return to the **Cloud Shell** window.
- Preview the web application by clicking on the Web Preview button and selecting **Preview on port 8080**.
- Click **Take Test**.
- Click **Places**.
- Complete the question.
- Rate the quiz, enter some feedback, and then click **Send Feedback**.

![d](https://user-images.githubusercontent.com/73010204/143730336-cd7dec47-b5e4-4047-8d27-96fd12d60024.png)</br>
![e](https://user-images.githubusercontent.com/73010204/143730339-b159799b-5201-4d2f-8976-24ca2418f458.png)</br>

### View Cloud Function monitoring and logs
- Return to the **Cloud Platform Console**, on the **Cloud Functions** page.
- Click _process-feedback_. You should see a monitoring graph for Cloud Function invocations. It takes a few minutes for the graph to show the invocation of your function
- Click **Logs** in the upper right of the page.
![g](https://user-images.githubusercontent.com/73010204/143730555-b5edbffd-436f-40e6-8ba9-0942ac18e084.PNG)
- If your Cloud Pub/Sub message isn't displayed, click **Jump to now**.
- Go to Cloud Shell to also see your feedback.
![h](https://user-images.githubusercontent.com/73010204/143730556-9e0083c4-e912-485d-8127-26552805eb47.PNG)</br>

### Examining the case study application code
In the **Cloud Platform Console**, click **Launch code editor**.</br>
- Navigate to **cloudfunctions/start**.
- Select the **index.js** file in the **...function** folder. 
- Select the **package.json** file. This file contains the list of dependencies that this function needs to run. Cloud Functions automatically install the dependencies. 
- Select the **languageapi.js** file. This file contains code to process feedback text and return the Natural Language ML API sentiment score.
- Select the **spanner.js** file. This file contains code to insert a record into a Cloud Spanner database.

### Coding a Cloud Function
In this section you write the code to create a Cloud Function that retrieves the Cloud Pub/Sub message data, invokes the Natural Language ML API to perform sentiment detection, and inserts a record into Cloud Spanner.</br>
Write code to modify a Cloud Function
- Return to the _...function/index.js_ file.
- Load the _languageapi_ and _spanner_ modules. These modules are in the same folder as the **index.js** file.
- In the _subscribe()_ method, after the existing code that loads the Cloud Pub/Sub message into a buffer, convert the PubSub message into a feedback object by parsing it as JSON data.
- Return a promise that invokes the _languageapi_ module's _analyze_ method to analyze the feedback text.
- Chain a _.then(...)_ method to the end of the return statement.
- Supply an arrow function as the value of the callback.
- In the body of the arrow function, log the Natural Language API sentiment score to the console.
- Add a new property called score to the feedback object.
- Complete the arrow function body by returning the feedback object.
- Chain a second _.then(...)_ method to the end of the first one. This one uses the _spanner_ module to save the feedback.
- Write a third _.then(...)_ chained method, including an arrow function with no arguments and an empty body as the value of the callback.
- In the body of this callback, log a message to indicate that the feedback has been saved, and return a success message.
- Attach a _.catch(...)_ handler to the end of the chain, which logs the error message to the console.

**function/index.js**
```sh
// TODO: Load the ./languageapi module
const languageAPI = require('./languageapi');
// END TODO
// TODO: Load the ./spanner module
const feedbackStorage = require('./spanner');
// END TODO
exports.subscribe = function subscribe(event) {
  // The Cloud Pub/Sub Message object.
  // TODO: Decode the Cloud Pub/Sub message
  // extracting the feedbackObject data
  // The message received from Pub/Sub is base64 encoded, and
  // the data submitted by students is in a data property     
  const pubsubMessage = Buffer.from(event.data, 'base64').toString();
  let feedbackObject = JSON.parse(pubsubMessage);
  console.log('Feedback object data before Language API:' + JSON.stringify(feedbackObject));
  // END TODO
  // TODO: Run Natural Language API sentiment analysis
  // The analyze(...) method expects to be passed the
  // feedback text from the feedbackObject as an argument,
  // and returns a Promise.
  return languageAPI.analyze(feedbackObject.feedback).then(score => {
    // TODO: Log the sentiment score
  	console.log(`Score: ${score}`);
    // END TODO
    // TODO: Add new score property to feedbackObject
  	feedbackObject.score = score;
    // END TODO
    // TODO: Pass feedback object to the next handler
  	return feedbackObject;
    // END TODO
  })
  // TODO: insert record
  .then(feedbackStorage.saveFeedback).then(() => {
  	// TODO: Log and return success
    console.log('feedback saved...');
  	return 'success';
    // END TODO
  })
  // END TODO
  // TODO: Catch and Log error
  .catch(console.error);
  // End TODO
};
```
Save the file.
### Package and deploy the Cloud Function code
Return to **Cloud Shell** and stop the web application by pressing **Ctrl+C**.</br>
To change the working directory to the Cloud Function code, enter the following command:
```sh
cd function
```
To zip up the files needed to deploy the function, enter the following command:
```sh
zip cf.zip *.js*
```
This generates a zip archive named cf.zip that includes all the JavaScript and JSON files in the folder.</br>
To stage the zip file into Cloud Storage, enter the following command:
```sh
gsutil cp cf.zip gs://$GCLOUD_BUCKET/
```
This copies the zip archive into the Cloud Storage bucket named after your project ID with a _-media_ suffix.</br>
Return to the **Cloud Platform Console** and navigate to **Cloud Functions**.</br>
Select the _process-feedback_ function.</br>
Click **Edit**.</br>
Click **Next**.</br>
Under **Source code**, select **ZIP from Cloud Storage**.</br>
For **Cloud Storage location**, Click **Browse**, select the **cf.zip** file in the bucket named after your GCP project ID with the _-media_ suffix, and click **Select**.</br>
In the **Entry point** field, type **subscribe**.</br>
Click **Deploy**.</br>
![i](https://user-images.githubusercontent.com/73010204/143730886-20423fc4-fcc4-43b2-9a2b-6387dd49313e.png)

## Testing the case study application
Run the web application
- Return to the **Cloud Shell** window.
- Change the working folder back to the start folder for the **cloudfunctions** lab.
```sh
cd ..
```
- To start the web application, execute the following command:
```sh
npm start
```
- Preview the web application.
- Click **Take Test**.
- Click **Places.**
- Complete the question.
- Enter some feedback, and click **Send Feedback**.

![j](https://user-images.githubusercontent.com/73010204/143730981-cad948d7-3c3d-485f-a5b2-45121da38f22.png)

### View Cloud Function monitoring and logs
Return to the **Cloud Functions** section of the **Cloud Platform Console**.</br>
Click the function name, _process-feedback_. You should see a monitoring graph for Cloud Function invocations. It takes a few minutes for the graph to show the invocation of your function.</br>
![k](https://user-images.githubusercontent.com/73010204/143731018-1e339835-58db-4f86-aec3-72e9cc579fef.PNG)</br>

Click **Logs**. If your Cloud Pub/Sub message isn't displayed, click **Jump to now**. Refresh the logs every few minutes until the log entries show that your function executed.</br>
![l](https://user-images.githubusercontent.com/73010204/143731053-ef7ed0ca-3684-412e-a75e-46f831e996df.PNG)</br>

### View Cloud Spanner data
On the **Navigation menu**, click **Spanner**.</br>
Click **Quiz instance**, and then on the left pane, **quiz-database > Query**.</br>
Run the following query:
```sh
SELECT * FROM Feedback
```
You should see that a new record has been added to the Feedback table.</br>
![m](https://user-images.githubusercontent.com/73010204/143731089-21b7f7e4-8799-4924-aaf7-a971d3820b88.png)
--------------------------------------------
## Congratulations!
We did
- Create a Cloud Function that responds to Cloud Pub/Sub messages.
- Deploy multiple files to a Cloud Function.

## Referenence
**Google Cloud Fundamentals: Securing and Integrating Components of your Application** course
https://www.coursera.org/learn/securing-integrating-components-app/home/welcome
