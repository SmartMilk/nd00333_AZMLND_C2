# Udacity Nanodegree Machine Learning Engineer - Project 2 (Operationalizing Machine Learning)

This project involves the use of data from the [UCI Bankmarketing dataset](https://archive.ics.uci.edu/ml/datasets/Bank%2BMarketing) that is used to train a ML model via Azure's AutoML module to predict whether a potential client will provide a term deposit; the model is consumed via a REST endpoint. I further demonstrate the consumption and benchmarking capabilities of ML model REST endpoints using additional Azure, Swagger and Apache functionality. Additionally, I showcase how ML Pipelines in Azure can be created and consumed. Both elements feed into the core design of Machine Learning Operations: Trained ML models are matured to the point where they can be consumed by other users with their own data, while the pipelines showcase automated steps in ML processes that can for example be shared among a team of data scientists to optimize ML processes. 

## Architectural Diagram
![Project Architecture](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/Udacity%20Project%202%20Architecture.drawio.png "Architecture")

The above image represents the architecture of the key services deployed for this project. The source data is the bankmarketing dataset from the UCI repository. In brief, the dataset contains a number of variables that may or may not have an influence in predicting whether a client will "subscribe a term deposit". 

Through Microsoft Azure, the dataset is downloaded and passed to the AutoML module. The AutoML module employs a number of classifiers with variations in hyperparameters, and undergoes a proprietary process to select and fine-tune a ML model best suited to predict the outcome. Results elaborated further in the "Key Steps" section.

Once the AutoML module completes its run, the best model is selected and deployed as a webservice endpoint via Azure Container Instance. From there, the endpoint can be consumed in a number of ways. The three consumption methods highlighted in this architecture are:

* Swagger. Swagger provides an API to auto generate high-quality, easy to digest documentation for the endpoint, allowing users to understand how they need to format their input data in order to be fed into the endpoint.

* Application Insights. This is an Azure supported service that allows endpoint creators to monitor how their endpoint is performing, and whether there needs to be changes made to the API for acceptable service. For example, an influx of http 503 errors indicate there is insufficient amount of resources to handle incoming API requests and the user may need to upgrade the amount of computing resources available for the API. 

* Apache Benchmarking. When deploying the endpoint for the first time, it is useful to employ benchmarking services such as the aforementioned Apache Benchmarking tool to "stress test" the endpoint, and ensure that the endpoint is capable of handling the expected volume of API calls

## Key Steps
The following key steps describe how the architecture in the previous session is constructed, with proof that I was able to put these services into successful deployment. Additionally, there is a discussion (and image proof) on how the training of the bankmarketing data can be automated via publishing a pipeline process as an endpoint. 

### AutoML Model Generation

The bankmarketing dataset is downloaded from a static URL and formatted as an Azure dataset, as seen in the screenshot below.

![bankmarketing-dataset](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/bankmarketing_dataset_register_proof.jpg)

Next, Azure's AutoML module is utilized to create a best-fit module for the bankmarketing data. The AutoML module ranks model performance of classifiers for the predicting variable "y" (whether a term deposit will happen or not) using an AUC weighting factor. In the screenshot below, the AutoML module takes approximately 20-25 minutes to complete with a DS2_V2 remote computing cluster.

![automl-complete](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/automl_experiment_completed.jpg)

Further proof of the bankmarketing dataset being used in the AutoML module is shown below:

![bankarmet-automl-proof](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/bankmarketing_with_automl_module_proof.jpg)

Exploring the 'Models' section of the completed AutoML run reveals that a Voting Ensemble model is the best performing classifier, with an AUC-weighted score of 0.947, indicating the model has good discriminitave power and can predict either of the two outcomes with high accuracy. 

### Endpoint deployment

![automl-best-model](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/automl_run_best_model_verbose.jpg)

With a satisfactory model produced, a webservice endpoint is created using Azure's UI. The webservice is configured as an Azure Container Instance with a maximum timeout factor of 60 seconds as per Azure standards. After roughly 20 minutes, the endpoint is deployed and running in a healthy state, meaning that is suitable for consumption.

### Deploying Application Insights

![healthy-endpoint](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/endpoint_active_and_healthy.jpg)

To monitor the health of my deployed endpoint, Application Insights can be automatically enabled through the Azure UI when initially deploying the model, however it can also be enabled post-deployment through the Azure SDK. Screenshots demonstrating the successful deployment of Application Insights via the SDK in a python script and its corresponding proof on Azure are shown in the screenshots below:

![AppIn-SDK-Deployment](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/successful_deployment_of_application_insights.jpg)

Note how executing the Python script 'logs.py' is the process to trigger the enabling of Application Insights

![AppIn-Web-Screenshot](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/app_insight_enabled_via_SDK.jpg)

The generated URL in the above screenshot can then be used to monitor how the endpoint is faring over a period of time.

### Generating Swagger Documentation

Next, the endpoint API is formatted for Swagger documentation. This is performed simply by bashing a shell script "shell.sh" to employ Swagger in a localhost port range, which is chosen to run from 8080-9000 in this particular example. The documention is then generated by passing a JSON config file autogenerated by the endpoint in Azure into the Swagger API, which is executed in a python script 'serve.py'. The resulting documentation is shown in the screenshot below.

![Swagger-UI](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/swagger_documentation_screenshot.jpg)

### Endpoint Consumption Test

To make sure the endpoint is operating correctly, 2 datapoints in a JSON style format are fed into the API via a Python script 'endpoint.py'. The endpoint is operating correctly if it produces a binary response on the command line. In this instance, the first datapoint is a 'yes' and the second datapoint is a 'no', indicating one client would subscribe a term deposit while the other would not. Note that in my screenrecording, both results are a 'no', which is due to using different lab sessions thus a slightly different AutoML model is chosen. 

![Endpoint-consumption](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/endpoint_consumption_proof.jpg) 

### Apache Benchmarking

The endpoint undergoes some additional testing to prove that the API can handle a high-request volume. This is achieved through the Apache Benchmarking API. Using preset configs, the API is tested 10 times using a JSON packet file containing 2 test points of data. The below screenshot shows the output of the benchmarking process. The key figure to note is the average time of request, which is shown to be 100ms. While this is acceptable response time, it should be considered the input JSON packet is only several kB large and has 2 testing points of data, which it is not indicative of how well the API will perform with larger data files. 

![Apache](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/apache_benchmarking_proof.jpg)

### Pipeline Endpoints

To automate the training of the bankmarketing dataset for future automation, a pipeline is created and submitted as an endpoint using an Azure Jupyter notebook script. The following screenshots show the pipeline being created, scheduled with the RunDetails Widget, and deposited as an active endpoint in Azure respectively.

![Pipeline-creation](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/pipeline_creation_proof.jpg)

![Pipeline-widget](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/pipeline_rundetails_widget_proof.jpg)

![Pipeline-endpoint](https://github.com/SmartMilk/nd00333_AZMLND_C2/blob/master/starter_files/project_images/pipeline_endpoint_active_proof.jpg)

## Screen Recording
[Click this link to go to youtube video of the screencast showcase for this project.](https://www.youtube.com/watch?v=sMb0tTM2qJw)

## Standout Suggestions
To improve the endpoint service, some possible suggestions are:
* Utilize the Application Insights service to email the endpoint owner when there is a large influx of errors, this will alert the owner to the problem and fix the issue with the endpoint faster than scheduled/random health checks. 

* More rigorous testing of the API for high-demand loads by using Apache with larger file sizes. If the load becomes too much for the endpoint, some additional pipeline work could be deployed, for example - including an additional cluster that is only spun-up to handle high-demand workloads. 
