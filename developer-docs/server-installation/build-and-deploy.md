---
title: Build and Deploy
page_title: Build and Deploy
description: Explains the build and deploy jobs for Sunbird
keywords: build, deploy, install, sunbird
allowSearch: true

---

## Overview

To install Sunbird `release-3.8.0`, we will need to run many Jenkins jobs and a couple of manual steps. Some of the jobs need to be run in a specific order. This page details out the list of jobs, their order and the github release tags to be used for each job.

### Installing Sunbird

#### Create Plugin Containers / Folder Structure

- We will create few folders in azure for storing sunbird plugins
- Our deployment job always tries to delete the folder first before uploading, so we need to have the placeholders before running the jobs
- We will create the folders by uploading random files from a random folder on your PC
- Create a public container with the name you have set for the variable `sunbird_content_azure_storage_container` and run the below commands

```bash
az storage blob upload-batch --destination sunbird_content_azure_storage_container/collection-editor --source some_folder --account-name storage_account_name --account-key storage_account_key

az storage blob upload-batch --destination sunbird_content_azure_storage_container/generic-editor --source some_folder --account-name storage_account_name --account-key storage_account_key

az storage blob upload-batch --destination sunbird_content_azure_storage_container/content-editor --source some_folder --account-name storage_account_name --account-key storage_account_key

az storage blob upload-batch --destination sunbird_content_azure_storage_container/v3/preview --source some_folder --account-name storage_account_name --account-key storage_account_key
```

- Upload the default T&C file to `sunbird_content_azure_storage_container` container in a folder named **terms-and-conditions**. You can get a copy of the sample T&C file from [here](https://sunbirdpublic.blob.core.windows.net/installation/terms-and-conditions/terms-and-conditions-v9.html). You are free to edit the HTML as per your requirements
- Create a private container named `label` in Azure
- Run the below command in your Jenkins VM where you cloned the sunbird-devops repo

```bash
cd sunbird-devops/utils/portal
az storage blob upload-batch --destination label --source labels --account-name storage_account_name --account-key storage_account_key
```

#### Upload Initial Plugins

- Upload the initial set of plugins to `sunbird_content_azure_storage_container` container
- Download the initial Plugins from [here](https://sunbirdpublic.blob.core.windows.net/installation/content-plugins.zip)
- Run the below command from the directory where **content-plugins.zip** is unzipped

```bash
az storage blob upload-batch --destination sunbird_content_azure_storage_container/content-plugins --source content-plugins --account-name storage_account_name --account-key storage_account_key
```

- Create a container named `public` in your Azure storage account from the Azure portal with public access level
- Create a container named `artifacts` in your Azure storage account from the Azure portal with private access level


#### Upload Maxmind Database to Azure Blob

- Download the Maxmind city database in zip format from Maxmind website
- Upload the zip file to the `artifacts` container in Azure

#### Code Builds

> **Note**:
>
>- Jenkins will have many jobs which may not be documented or is not suitable for adoption yet
>- Jobs listed on this page is sufficient to setup Sunbird and not every job present in Jenkins is required to be run
>- Jobs can be run in parallel to speed up execution. The only exception is you should **NOT** run multiple jobs which clone the code from same repo in parallel
>- If you get errors in some of the build jobs, rerun the job again
>- The errors are usually due to missing maven jars or maven repo timeouts
>- If you get errors even after re-running the job, come back to it later, there are other jobs which would generate dependent jars

|Jenkins Job to Run|Github Tag|Github Repo|Comments|
|------------------|----------|-----------|--------|
|Build/Core/AdminUtils|release-3.7.0_RC1|<https://github.com/project-sunbird/sunbird-apimanager-util.git>|Handles mobile device registration, refresh token|
|Build/Core/Analytics|release-3.8.0_RC3|<https://github.com/project-sunbird/sunbird-analytics-service.git>|Location Register, Location Retrieval|
|Build/Core/APIManager|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|The API Gateway|
|Build/Core/ApiManagerEcho|master-RC1|<https://github.com/project-sunbird/sunbird-echo-service.git>|Simply echos the request|
|Build/Core/Assessment|release-3.8.0_RC9|<https://github.com/project-sunbird/knowledge-platform.git>|Handles course assessments|
|Build/Core/Bot|release-3.8.0_RC2|<https://github.com/project-sunbird/sunbird-bot.git>|Frontend for the chatbot|
|Build/Core/Cassandra|release-3.8.0_RC2|<https://github.com/project-sunbird/sunbird-utils.git>|Handles the Cassandra database schema|
|Build/Core/CassandraTrigger|release-3.8.0_RC2|<https://github.com/project-sunbird/sunbird-utils.git>|Creates few Cassandra Triggers|
|Build/Core/Cert|release-3.5.0_RC2|<https://github.com/project-sunbird/cert-service.git>|Handles course certificates|
|Build/Core/CertRegistryService|release-3.4.0_RC1|<https://github.com/project-sunbird/certificate-registry.git>|Helper for Certificate service|
|Build/Core/Content|release-3.8.0_RC9|<https://github.com/project-sunbird/knowledge-platform.git>|Handles the Contents and Content Metadata
|Build/Core/Dial|release-3.7.0_RC2|<https://github.com/project-sunbird/sunbird-dial-service.git>|Handles QR code generation|
|Build/Core/DiscussionsMiddleware|release-3.8.0_RC2|<https://github.com/Sunbird-Ed/discussions-middleware.git>|Middleware for discussion forum|
|Build/Core/EncService|release-3.8.0_RC1|<https://github.com/project-sunbird/enc-service>|Encryption service to setup certificate signing keys|
|Build/Core/Groups|release-3.8.0_RC1|<https://github.com/project-sunbird/groups-service.git>|Handles groups functions|
|Build/Core/Keycloak|release-3.8.0_RC1|<https://github.com/project-sunbird/sunbird-auth.git>|User authentication service|
|Build/Core/KnowledgeMW|release-3.8.0_RC3|<https://github.com/project-sunbird/knowledge-mw-service.git>|Middleware for Content service|
|Build/Core/Learner|release-3.8.0_RC31|<https://github.com/project-sunbird/sunbird-lms-service.git>|Handles user and organizations|
|Build/Core/Lms|release-3.8.0_RC8|<https://github.com/project-sunbird/sunbird-course-service.git>|Handles courses|
|Build/Core/Nodebb|github_release_tag: release-3.8.0_RC1, nodebb_branch: v1.16.0|<https://github.com/project-sunbird/sunbird-nodebb.git>|Handles the discussion forum|
|Build/Core/Notification|release-3.7.0_RC3|<https://github.com/project-sunbird/sunbird-notification-service.git>|Handles notifications|
|Build/Core/Player|release-3.8.2_RC2|<https://github.com/Sunbird-Ed/SunbirdEd-portal.git>|Handles the UI elements|
|Build/Core/Print|release-3.8.0_RC1|<https://github.com/project-sunbird/print-service.git>|Handles PDF generation of certificates|
|Build/Core/Proxy|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Nginx web server / proxy|
|Build/Core/Report|release-3.2.0_RC1|<https://github.com/project-sunbird/sunbird-report-service.git>|Handles some of the reports|
|Build/Core/Router|release-3.8.0_RC2|<https://github.com/project-sunbird/sunbird-bot.git>|Backend for chatbot|
|Build/Core/Search|release-3.8.0_RC9|<https://github.com/project-sunbird/knowledge-platform.git>|Handles contents / course search|
|Build/Core/Taxonomy|release-3.8.0_RC9|<https://github.com/project-sunbird/knowledge-platform.git>|Handles the taxonomies|
|Build/Core/Telemetry|release-3.3.0_RC1|<https://github.com/project-sunbird/sunbird-telemetry-service.git>|Handles the telemetry generated|
|Build/Core/Yarn|release-2.6.0|<https://github.com/project-sunbird/sunbird-lms-jobs.git>|Handles SSO and account merges|
|Build/KnowledgePlatform/CassandraTrigger|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Creates few Cassandra Triggers|
|Build/KnowledgePlatform/FlinkJobs|release-3.8.0_RC7|<https://github.com/project-sunbird/knowledge-platform-jobs.git>|Multiple functions like search indexing, video streaming|
|Build/KnowledgePlatform/Learning|github_release_tag: release-3.8.0_RC18, profile_id: platform-services|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Handles frameworks|
|Build/KnowledgePlatform/Neo4j|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Generates neo4j plugins|
|Build/KnowledgePlatform/SyncTool|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Generates a jar to manually sync contents |
|Build/KnowledgePlatform/Yarn|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Multiple functions like publishing contents, issuing certificates|
|Build/DataPipeline/AdhocScripts|release-3.8.0_RC9|<https://github.com/Sunbird-Ed/sunbird-data-products.git>|Generates supporting scripts for reporting|
|Build/DataPipeline/AnalyticsCore|release-3.8.0_RC4|<https://github.com/project-sunbird/sunbird-analytics-core.git>|Generates core libraries for reporting|
|Build/DataPipeline/CoreDataProducts|release-3.8.0_RC4|<https://github.com/project-sunbird/sunbird-core-dataproducts.git>|Generates supporting libraries for reporting|
|Build/DataPipeline/EdDataProducts|release-3.8.0_RC9|<https://github.com/Sunbird-Ed/sunbird-data-products.git>|Generates supporting libraries for reporting|
|Build/DataPipeline/ETLJobs|release-3.8.0_RC10|<https://github.com/Sunbird-Ed/sunbird-data-products.git>|Extract, Transform, Data Loading jobs|
|Build/DataPipeline/FlinkPipelineJobs|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Handles processing of telemetry data|
|Build/DataPipeline/Secor|secor-0.29_RC2|<https://github.com/project-sunbird/secor.git>|Handlles backup of telemetry data to Azure Blob|
|Build/Plugins/CollectionEditor|release-3.6.0_RC1|<https://github.com/project-sunbird/sunbird-collection-editor.git>|A bunch of plugins to handle collection editing|
|Build/Plugins/ContentEditor|release-3.6.0_RC2|<https://github.com/project-sunbird/sunbird-content-editor.git>|A bunch of plugins to edit certain types of content|
|Build/Plugins/ContentPlayer|release-3.8.1_RC1|<https://github.com/project-sunbird/sunbird-content-player.git>|A bunch of plugins to handle content playback|
|Build/Plugins/ContentPlugins|release-3.8.1_RC1|<https://github.com/project-sunbird/sunbird-content-plugins.git>|A bunch of base plugins to support the content editors and content player|
|Build/Plugins/GenericEditor|release-3.6.0_RC1|<https://github.com/project-sunbird/sunbird-generic-editor.git>|A bunch of plugins to edit certain types of content|

#### Infra Provision

> **Note**:
>
>- Jobs can be run in parallel to speed up execution. The only exception is you should **NOT** run multiple jobs which will provision packages on the same server in parallel

|Jenkins Job to Run|Github Tag|Github Repo|Comments|
|------------------|----------|-----------|--------|
|OpsAdministration/Core/Bootstrap|hosts: env, branch_or_tag: release-3.8.0_RC14, tag: bootstrap_any,node_exporter|<https://github.com/project-sunbird/sunbird-devops.git>|Creates the deployer user and installs node exporter and python packages for ansible|
|OpsAdministration/KnowledgePlatform/Bootstrap|hosts: env, branch_or_tag: release-3.8.0_RC14, tag: bootstrap_any,node_exporter|<https://github.com/project-sunbird/sunbird-devops.git>|Creates the deployer user and installs node exporter and python packages for ansible|
|OpsAdministration/DataPipeline/Bootstrap|hosts: env, branch_or_tag: release-3.8.0_RC14, tag: bootstrap_any,node_exporter|<https://github.com/project-sunbird/sunbird-devops.git>|Creates the deployer user and installs node exporter and python packages for ansible|
|Provision/Core/ApplicationElasticSearch|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Installs Elasticsearch used by the applications|
|Provision/Core/Cassandra|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Installs Cassandra database|
|Provision/Core/CassandraExporter|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Installs Cassandra prometheus exporter|
|Provision/Core/ESMapping|indices_name: all, branch_or_tag: release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates Elasticsearch indices and mappings|
|Provision/Core/Keycloak|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Installs java and other pre-requisites for Keycloak service|
|Provision/Core/LogES|release-3.9.0_RC16|<https://github.com/project-sunbird/sunbird-devops.git>|Installs Elasticsearch used to store application and VM logs|
|Provision/Core/MongodbCluster|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Installs Mongo database|
|Provision/Core/Postgres|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Installs Postgres database|
|Provision/Core/PostgresDbUpdate|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates Postgres tables, schema and users|
|Provision/KnowledgePlatform/Learning|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Install tomcat and other pre-requisites for Learning service|
|Provision/KnowledgePlatform/Neo4j|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Installs Neo4j database|
|Provision/KnowledgePlatform/Yarn|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Install Hadoop and other pre-requisites to run Yarn|
|Provision/DataPipeline/AnalyticsGeoLocationDBSetup|release-3.8.0_RC13, geoip_zip_csv_file_name: Enter the maxmind city database zip file name you uploaded to `artifacts` container|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Creates tables and schema in Postgres|
|Provision/DataPipeline/AnalyticsSpark|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Install Apache Spark and other pre-requisites for reporting|
|Provision/DataPipeline/Druid|release-3.8.0_RC13, service: Select All, remote: raw|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Installs Druid|
|Provision/DataPipeline/InfluxDB|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Install Influx DB used by monitoring system|
|Provision/DataPipeline/Kafka|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Installs Kafka|
|Provision/DataPipeline/KafkaIndexer|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Install Logstash to read telemetry data from Kafka and index into Log ES|
|Provision/DataPipeline/Postgres|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Installs Postgres database and users|
|Provision/DataPipeline/PostgresDbUpdate|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Creates Postgres tables, schema and users|
|Provision/DataPipeline/Redis|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Install Redis|
|Provision/DataPipeline/Zookeeper|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Installs Zookeeper|


#### ArtifactUpload

- Every job in the **Build** directory has a corresponding job in **ArtifactUpload** directory with the same name
- These jobs are auto triggered and usually run without issues after the corresponding job in the build directory succeeds 
- The job will fail if your ansible inventory setup is incorrect or incomplete
- If the job has failed, fix the ansible variables issue and rerun the job to upload the artifact / docker image
- Ensure the artifact upload jobs are successful before proceeding

#### Code Deploy

> **Note**:
>
>- We will run the jobs which are a pre-requisite for other jobs first
>- The pre-requisite jobs reside in different folders so we will be jumping across folders
>- Jobs in the **Deploy** directory can be run in parallel and don't have any restrictions if the code is from same repo unlike the **Build** directory
>- Ensure you don't run those jobs in parallel which modify the databases (such as cassandra, neo4j etc)

|Jenkins Job to Run|Github Tag|Github Repo|Comments|
|------------------|----------|-----------|--------|
|Deploy/Kubernetes/BootstrapMinimal|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates namespaces, configmaps and secrets|
|Deploy/DataPipeline/BootstrapMinimal|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Creates namespaces, configmaps and secrets|
|Deploy/Kubernetes/Monitoring|branch_or_tag: release-3.8.0_RC14, tag: all|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys the monitoring stack which can be accessed via DOMAIN/grafana/ post nginx-public-ingress deployment|
|Deploy/Kubernetes/CassandraTrigger|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Installs the Cassandra triggers|
|Deploy/KnowledgePlatform/CassandraTrigger|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Installs the Cassandra triggers|
|Deploy/Kubernetes/CassandraDBUpdate|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates / Updates Cassandra DB schema|
|Deploy/KnowledgePlatform/CassandraDbUpdate|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Creates / Updates Cassandra DB schema|
|Deploy/Kubernetes/Cassandra|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates / Updates Cassandra DB schema|
|Deploy/Kubernetes/KafkaSetup|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates / Updates Kafka Topics|
|Deploy/DataPipeline/KafkaSetup|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Creates / Updates Kafka Topics|
|Deploy/KnowledgePlatform/KafkaSetup|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Creates / Updates Kafka Topics|
|Deploy/KnowledgePlatform/Learning|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Deploys the Learning service on tomcat|
|Deploy/KnowledgePlatform/Neo4j|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Deploys the Neo4j plugins|
|Deploy/KnowledgePlatform/Neo4jDefinitionUpdate|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Creates / Updates the Neo4j database|
|Deploy/KnowledgePlatform/Yarn|release-3.8.0_RC18|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Deploys the Samza jobs|
|Deploy/KnowledgePlatform/FlinkJobs|branch_or_tag: release-3.8.0_RC18, job_names_to_deploy: Select All|<https://github.com/project-sunbird/sunbird-learning-platform.git>|Deploys the Flink jobs in Kubernetes cluster|
|Deploy/KnowledgePlatform/LoggingFileBeatsVM|branch_or_tag: release-3.8.0_RC14, hosts: Select All, tags: default|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys filebeat in all the selected VMs|
|Deploy/Kubernetes/nginx-private-ingress|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys nginx with a private load balancer|
|Deploy/Kubernetes/APIManager|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Kong, the API gateway |
|Deploy/Kubernetes/OnboardAPIs|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates the APIs in Kong|
|Deploy/Kubernetes/OnboardConsumers|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates the consumers in Kong. Now read the comment mentioned in **Core/Secrets.yml** against the variable `core_vault_sunbird_api_auth_token`|
|Deploy/Kubernetes/Keycloak|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Keycloak|
|Deploy/Kubernetes/KeycloakRealm|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates the Sunbird Realm. Now read the comment mentioned in **Core/Secrets.yml** against the variables `core_vault_sunbird_sso_publickey`, `adminutil_refresh_token_public_key_kid`, `adminutil_refresh_token_secret_key`|
|Deploy/Kubernetes/BootstrapMongodb|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates the initial token in Mongo database|
|Deploy/Kubernetes/CertTemplate|branch_or_tag: release-3.8.0_RC14, sunbird_util_branch_or_tag: release-3.8.0_RC2|<https://github.com/project-sunbird/sunbird-devops.git>|Uploads the certificate templates into Azure blob|
|Deploy/Kubernetes/UploadChatbotConfig|branch_or_tag: release-3.8.0_RC14, bot_repo_branch: master|<https://github.com/project-sunbird/sunbird-devops.git>|Uploads the chatbot configs to Azure blob|
|Deploy/Kubernetes/UploadFAQs|branch_or_tag: release-3.9.0_RC16, source_folder: Select All|<https://github.com/project-sunbird/sunbird-devops.git>|Uploads the FAQs to Azure blob|
|Deploy/Kubernetes/UploadSchema|branch_or_tag: release-3.8.0_RC14, kp_branch_or_tag: release-3.8.0_RC9|<https://github.com/project-sunbird/sunbird-devops.git>|Uploads the Content schemas to Azure blob|
|Deploy/Kubernetes/AdminUtils|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Adminutils service|
|Deploy/Kubernetes/Analytics|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Analytics service|
|Deploy/Kubernetes/APIManagerEcho|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Echo service|
|Deploy/Kubernetes/Assessment|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Assessment service|
|Deploy/Kubernetes/Bot|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Bot service|
|Deploy/Kubernetes/Cert|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Cert service|
|Deploy/Kubernetes/CertRegistry|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys CertRegistry service|
|Deploy/Kubernetes/Content|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Content service|
|Deploy/Kubernetes/Dial|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Dial service|
|Deploy/Kubernetes/Enc|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Enc service|
|Deploy/Kubernetes/Groups|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Groups service|
|Deploy/Kubernetes/KnowledgeMW|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys KnowledgeMW service|
|Deploy/Kubernetes/Learner|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Learner service|
|Deploy/Kubernetes/Lms|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Lms service|
|Deploy/Kubernetes/Notification|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Notification service|
|Deploy/Kubernetes/Player|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Player service|
|Deploy/Kubernetes/Print|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Print service|
|Deploy/Kubernetes/Report|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Report service|
|Deploy/Kubernetes/Router|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Router service|
|Deploy/Kubernetes/Search|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Search service|
|Deploy/Kubernetes/Taxonomy|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Taxonomy service|
|Deploy/Kubernetes/Telemetry|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Telemetry service|
|Deploy/Kubernetes/Yarn|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys Samza jobs|
|Deploy/Kubernetes/LoggingFileBeatsVM|branch_or_tag: release-3.8.0_RC14, hosts: Select All, tags: default|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys filebeat in all the selected VMs|
|Deploy/Kubernetes/Logging|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Choose one **chartname** each time - `fluent-bit`, `kibana`, `oauth2-proxy`. Deploys Fluent Bit, Kibana and Oauth2 Proxy respectively. Kibana can accessed via DOMAIN/dashboard/|
|Deploy/Kubernetes/nginx-public-ingress|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys the nginx web server. Now you can open visit your domain|
|Deploy/Kubernetes/Nodebb|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys NodeBB service. Now read the comment mentioned in **Core/Secrets.yml** against the variable `discussionsmw_nodebb_authorization_token`|
|Deploy/Kubernetes/DiscussionsMW|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys DiscussionsMW service|
|Deploy/Plugins/CollectionEditor|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys CollectionEditor files to Azure blob|
|Deploy/Plugins/ContentEditor|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys ContentEditor files to Azure blob|
|Deploy/Plugins/ContentPlayer|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys ContentPlayer files to Azure blob|
|Deploy/Plugins/ContentPlugins|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys ContentPlugins files to Azure blob|
|Deploy/Plugins/GenericEditor|release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys GenericEditor files to Azure blob|
|Deploy/DataPipeline/AdhocScripts|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys AdHoc scripts in Spark VM|
|Deploy/DataPipeline/AnalyticsCore|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys jars in Spark VM|
|Deploy/DataPipeline/CoreDataProducts|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys jars in Spark VM|
|Deploy/DataPipeline/DruidIngestion|branch_or_tag: release-3.9.0_RC12, ingestion_task_names: Select All|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys and starts Druid ingestion tasks. **This job will be updated in a couple of days. Please wait until then**|
|Deploy/DataPipeline/EdDataProducts|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys jars in Spark VM|
|Deploy/DataPipeline/ETLJobs|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys ETL scripts in Spark VM|
|Deploy/DataPipeline/ETLDruidContentIndexer|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys ETLDruidContentIndexer|
|Deploy/DataPipeline/ETLUserCacheIndexer|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys ETLUserCacheIndexer|
|Deploy/DataPipeline/Secor|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys Secor Statefulsets in Kubernetes cluster|
|Deploy/DataPipeline/FlinkPipelineJobs|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys Flink jobs Kubernetes cluster|
|Deploy/DataPipeline/GraphitePrometheusExporter|release-3.8.0_RC13|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Deploys graphite prometheus exporter in Druid VM|
|Deploy/Kubernetes/LoggingFileBeatsVM|branch_or_tag: release-3.8.0_RC14, hosts: Select All, tags: default|<https://github.com/project-sunbird/sunbird-devops.git>|Deploys filebeat in all the selected VMs|

#### Post Installation Steps

|Jenkins Job to Run|Github Tag|Github Repo|Comments|
|------------------|----------|-----------|--------|
|OpsAdministration/Core/PostInstallScript|branch_or_tag: release-3.8.0_RC14|<https://github.com/project-sunbird/sunbird-devops.git>|Creates the default forms, framework, users, channel, licenses etc. Please ensure you provide all the values that the job requires. You need to also ensure the script is successful by closely inspecting the output line by line on the Jenkins console log. You can also take a look at the script and API's and create your own data if you don't require the default values.|
|Deploy/DataPipeline/AnalyticsPopulatePSQLConsumerChannelMapping|release-3.8.0_RC13, channel_id: your sunbird organisation id, consumer_id: kong consumer id|<https://github.com/project-sunbird/sunbird-data-pipeline.git>|Adds kong consumer in postgres Analytics DB to whitelist some of the API's. You can get the kong cosumer id by querying in postgres on kong db `select * from consumers where username = 'api-admin';`|
