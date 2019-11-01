---
layout: post
title:  "VPC and NAT"
date:   2019-10-15 11:22:22 +0900
categories: cloud network
---
Data ingestion from server

Stackdriver
- 로깅
- 모니터링

vs elastic search (+kibana)

비용절감.

워크샵에서는 qwiklab이라는 시스템을 이용해서 실습을 진행한다. 실습용 profile을 받고, 해당 profile의 cloud console에 접속하여 진행하기 때문에 해당 실습이 종료되면 profile 삭제등으로 크레딧이 청구될 일이 없다는 장점이 있다. 실제로 다른 워크샵에서는 리소스를 깨끗히 정리하지 못해서 비용이 청구되는 일이 굉장히 많은데, 이 부분을 깔끔히 해결했다.

이 예제에서는 쿠버네티스 엔진 클러스터를 만들고, 간단한 샘플 어플리케이션을 배포한다. 기본적으로 GCP의 쿠버네티스 엔진 클러스터는 Terraform 을 이용해서 일단 application을 Kubernetes cluster에 배포한다. 
> *Note*: Terraform이란 무엇?
> - 테라폼은 오픈소스 인프라 스트럭처 관리 도구이다. Chef나 Ansible과 같은 Provisoning 
세 가지의 terraform 파일을 사용하는데, main.tf, provider.tf, variables.tf 이다. Terraform은 특정 디렉토리의 모든 tf 확장자 파일을 읽어들여 리소스 생성/수정/삭제를 진행하므로, 
어떤식으로 tf 파일을 작성할지에 대해서 제한은 없지만, 이 튜토리얼에서는 3개로 나눠서 진행하였다. 먼저, provider.tf는 이름에서도 알수 있듯이, 

For instance, Terraform can build out GCP projects and compute instances, etc., even set up a Kubernetes Engine cluster and deploy applications to it. When requirements change, the descriptor can be updated and Terraform will adjust the cloud infrastructure accordingly.
예를 들어, 테라폼은 

-----------------
Deployment
Following the principles of Infrastructure as Code and Immutable Infrastructure, Terraform supports the writing of declarative descriptions of the desired state of infrastructure. When the descriptor is applied, Terraform uses GCP APIs to provision and update resources to match. Terraform compares the desired state with the current state so incremental changes can be made without deleting everything and starting over. For instance, Terraform can build out GCP projects and compute instances, etc., even set up a Kubernetes Engine cluster and deploy applications to it. When requirements change, the descriptor can be updated and Terraform will adjust the cloud infrastructure accordingly.

This example will start up a Kubernetes Engine cluster and deploy a simple sample application to it. By default, Kubernetes Engine clusters in GCP are provisioned with a pre-configured Fluentd-based collector that forwards logs to Stackdriver. Interacting with the sample app will produce logs that are visible in the Stackdriver Logging and other log event sinks.

Update the provider.tf file
Remove the provider version for the Terraform from the provider.tf script file.

Edit the provider.tf script file.

nano ~/gke-logging-sinks-demo/terraform/provider.tf

If the file contains static version string for the google provider as given below then remove it.

....
provider "google" {
  project = var.project
  version = "~> 2.10.0"
}

After modification your provider.tf script file should look like:

...
provider "google" {
  project = var.project
}

Save the file.

Deploying the cluster
There are three Terraform files provided with this lab example. The first one, main.tf, is the starting point for Terraform. It describes the features that will be used, the resources that will be manipulated, and the outputs that will result. The second file is provider.tf, which indicates which cloud provider and version will be the target of the Terraform commands--in this case GCP.

The final file is variables.tf, which contains a list of variables that are used as inputs into Terraform. Any variables referenced in the main.tf that do not have defaults configured in variables.tf will result in prompts to the user at runtime.

To build out the environment you can execute the following make command:

make create

Note: If you get deprecation warnings related to the zone varibale, please ignore it and move forward in the lab.
Test Completed Task
Click Check my progress to verify your performed task. If you have successfully deployed necessary infrastructure with Terraform, you will see an assessment score.

Use Terraform to set up the necessary infrastructure
Validation
If no errors are displayed during deployment, after a few minutes you should see your Kubernetes Engine cluster in the GCP Console.

Go to Navigation menu > Kubernetes Engine > Clusters to see the cluster with the sample application deployed.

To validate that the demo deployed correctly, run:

make validate

Your output will look like this:

make_valid.png

Now that the application is deployed to Kubernetes Engine you can generate log data and use Stackdriver and other tools to view it.

어플리케이션은 쿠버네티스 엔진 위에 배포되기 때문에 로그 데이터를 생성하고 스택드라이버나 다른 툴을 사용해서 조회할 수 있다.

콘솔에서 Networking section -> Network services로 가면 기본 TCP LB 가 셋업되있다. LB 상세보기를 선택하면 Frontend라고 써있고 IP 주소가 나와있다. 해당 주소를 복사해서 브라우저에 붙여넣어보면, 아까 테라폼으로 띄웠던 샘플 어플리케이션이 나온다. 브라우저에서 어플리케이션을 열때마다, Stackdriver logging에 로그 이벤트 들을 publish한다. 몇번 리프레쉬를 해줘서, 로그를 조금 쌓아주도록 하자.

Logs in Stackdriver
Stackdriver provides a UI for viewing log events. Basic search and filtering features are provided, which can be useful when debugging system issues. The Stackdriver Logging is best suited to exploring more recent log events. Users requiring longer-term storage of log events should consider some of the tools you'll explore in the following sections.

To access the Stackdriver Logging console perform the following steps:

In the GCP Console, from the Navigation menu, in the Stackdriver section, click on Logging.
On this page change the Audited Resource filter to Kubernetes Container > stackdriver-logging > default. stackdriver-logging is the cluster; and default is the namespace).
Stack_clustername_default.png

On this screen, you can expand the bulleted log items to view more details about a log entry.
Stackdriver_GKE_error-log.png

On the logging console, you can perform any type of text search, or try out the various filters by log type, log level, timeframe, etc.

Viewing Log Exports
The Terraform configuration built out two Log Export Sinks. To view the sinks perform the following steps:

you should still be on the Stackdriver -> Logging page.

In the left navigation menu, click on the Exports menu option.

This will bring you to the Exports page. You should see two Sinks in the list of log exports.

You can edit/view these sinks by clicking on the context menu (three dots) to the right of a sink and selecting the Edit sink option.

Additionally, you could create additional custom export sinks by clicking on the Create Export option in the top of the navigation window.

Logs in Cloud Storage
Log events can be stored in Cloud Storage, an object storage system suitable for archiving data. Policies can be configured for Cloud Storage buckets that, for instance, allow aging data to expire and be deleted while more recent data can be stored with a variety of storage classes affecting price and availability.

The Terraform configuration created a Cloud Storage Bucket named stackdriver-gke-logging- to which logs will be exported for medium to long-term archival. In this example, the Storage Class for the bucket is defined as Nearline because the logs should be infrequently accessed in a normal production environment (this will help to manage the costs of medium-term storage). In a production scenario, this bucket may also include a lifecycle policy that moves the content to Coldline storage for cheaper long-term storage of logs.

To access the Stackdriver logs in Cloud Storage perform the following steps:

In the GCP Console from the Navigation menu click Storage.
Find the Bucket with the name stackdriver-gke-logging-<random-Id>, and click on the name.
You should see a list of directories corresponding to pods running in the cluster (e.g. autoscaler, dnsmasq, etc.).
Cloud Storage Bucket

You can click into any of the folders to browse specific log details like heapster, kubedns, sidecar, etc.

Logs in BigQuery
Stackdriver log events can be configured to be published to BigQuery, a data warehouse tool that supports fast, sophisticated, querying over large data sets.

The Terraform configuration will create a BigQuery DataSet named gke_logs_dataset. This dataset will be setup to include all Kubernetes Engine related logs for the last hour (by setting a Default Table Expiration for the dataset). A Stackdriver Export will be created that pushes Kubernetes Engine container logs to the dataset.

To access the Stackdriver logs in BigQuery, perform the following steps:

Note: The BigQuery Export is not populated immediately. It may take a few moments for logs to appear.

On the GCP console from Navigation menu, in the Big Data section, click on BigQuery.
In the left menu, click on your project name. You should see a dataset named gke_logs_dataset. Expand this dataset to view the tables that exist (Note: The dataset is created immediately, but the tables are what is generated as logs are written and new tables are needed).
Click on one of the tables to view the table details.
Review the schema of the table to note the column names and their data types. This information can be used in the next step when you query the table to look at the data.
BQ_logs_dataset.png

Click on Query Table towards the top right to perform a custom query against the table.
This adds a query to the Query Editor, but it has a syntax error.
Edit the query to add an asterisk (*) after Select to pull in all details from the current table. Note: A Select * query is generally very expensive and not advised. For this lab the dataset is limited to only the last hour of logs, so the overall dataset is relatively small.
Click Run to execute the query and return some results from the table.
The results window should display some rows and columns. You can scroll through the various rows of data that are returned. If you want, execute some custom queries that filter for specific data based on the results that were shown in the original query.

Test Completed Task
Click Check my progress to verify your performed task. If BigQuery sink written logs in BigQuery dataset, you will see an assessment score.

View Logs in BigQuery
Teardown
Qwiklabs will take care of shutting down all the resources used for this lab, but here’s what you would need to do to clean up your own environment to save on cost and to be a good cloud citizen:

make teardown

Since Terraform tracks the resources it created it is able to tear them all down.

Troubleshooting for your production environment
The install script fails with a Permission denied when running Terraform. 
The credentials that Terraform is using do not provide the necessary permissions to create resources in the selected projects. Ensure that the account listed in gcloud config list has necessary permissions to create resources. If it does, regenerate the application default credentials using gcloud auth application-default login.

Cloud Storage Bucket not populated 
Once the Terraform configuration is complete the Cloud Storage Bucket will be created, but it is not always populated immediately with log data from the Kubernetes Engine cluster. Give the process some time because it can take up to 2 to 3 hours before the first entries start appearing (https://cloud.google.com/logging/docs/export/using_exported_logs).

No tables created in the BigQuery dataset 
Once the Terraform configuration is complete the BigQuery Dataset will be created but it will not always have tables created in it by the time you go to review the results. The tables are rarely populated immediately. Give the process some time (minimum of 5 minutes) before determining that something is not working properly.

*Bigquery dataset에 테이블이 존재하지 않을때*
테라폼 설정이 완료되어


Congratulations
Enterprise-Customers.png stackdriver-logging-badge.png

Finish Your Quest
This self-paced lab is part of the Qwiklabs Google Kubernetes Engine Best Practices and Stackdriver Logging Quests. A Quest is a series of related labs that form a learning path. Completing a Quest earns you a badge to recognize your achievement. You can make your badge (or badges) public and link to them in your online resume or social media account. Enroll in either Quest and get immediate completion credit if you've taken this lab. See other available Qwiklabs Quests.

Next Steps / Learn More
Kubernetes Engine Logging
Viewing Logs
Advanced Logs Filters
Overview of Logs Exports
Procesing Logs at Scale Using Cloud Dataflow
Terraform Google Cloud Provider
Google Cloud Training & Certification
...helps you make the most of Google Cloud technologies. Our classes include technical skills and best practices to help you get up to speed quickly and continue your learning journey. We offer fundamental to advanced level training, with on-demand, live, and virtual options to suit your busy schedule. Certifications help you validate and prove your skill and expertise in Google Cloud technologies.