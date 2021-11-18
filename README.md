# Instrumenting containerized application services for AppDynamics business insights with Intersight Cloud Orchestrator (ICO)
## Contents
        Use Case

        Pre-requisites

        Intersight Target configuration for AppDynamics and on prem entities

        Provision infrastructure and deploy App services with AppDynamics instrumentation

            Step 1: Importing ICO template for App Services Deployment and Instrumentation

            Step 2: Importing ICO template for App Services Load Generation

            Step 3: Importing ICO template for App Services Decommissioning

            Step 4: Setup Tfiks-Global Variables

            Step 5: Setup Tfiks-Host Variables

            Step 6: Setup Tfiks-Rbac Variables

            Step 7: Setup Tfiks-Metrics Variables

            Step 8: Setup Tfiks-CluisterA Variables

            Step 9: Setup Tfiks-App Variables

            Step 10: Setup Tfiks-Load Variables

            Step 11: Setup Tfiks-Remove Variables

            Step 12: Execute ICO template for App Services Deployment and Instrumentation

            Step 13: Execute ICO template for App Services Load Generation

            Step 14: View AppDynamics Insights

        Undeploy applications and deprovision infrastructure



### Use Case

* As a DevOps and App developer, use ICO (Intersight Cloud Orchestrator) to enable existing containerized Java Micro services for AppDynamics Insights

* As DevOps and App Developer, use Intersight and AppDynamics to view app and infrastructure insights for Full Stack Observability

This use case addresses the fourth flow in the below diagram:

![alt text](https://github.com/prathjan/images/blob/main/tomflow4.png?raw=true)

### Pre-requisites

1. The VM template that you provision in Step 5 below will have a user "root/Cisco123" provisioned with sudo privileges. Terraform scripts will use this user credentials to remotely run installation scripts in the VM.

2. Sign up for a user account on Intersight.com. You will need Premier license as well as IWO license to complete this use case. 

3. Sign up for a TFCB (Terraform for Cloud Business) at https://app.terraform.io/. Log in and generate the User API Key. You will need this when you create the TF Cloud Target in Intersight.

4. You will need access to a vSphere infrastructure with backend compute and storage provisioned

5. You will also need an account in AppDynamics DevNet SAAS Controller (https://devnet.saas.appdynamics.com/) and should be able to log in and view your deployed Application insights.

6. You will need to have some minimal knowledge of Intersight ICO. Please review tutorials on Youtube as well as the following: https://intersight.com/help/saas/features/orchestration/configure#intersight_cloud_orchestrator

7. You will follow this codeexchange to set up your IKS cluster: https://developer.cisco.com/codeexchange/github/repo/CiscoDevNet/intersight-tfb-iks. Note the TFCB workspace for the IKS cluster provisioning, you will need this to proceed with this use case.

8. You will create an account on containers.cisco.com and save the username and password. This is to download the docker images from the repo.

### Intersight Target configuration for AppDynamics and on prem entities

You will log into your Intersight account and create the following targets. Please refer to Intersight docs for details on how to create these Targets:

    Assist

    vSphere

    AppDynamics

    TFC Cloud

    TFC Cloud Agent - When you claim the TF Cloud Agent, please make sure you have the following added to your Managed Hosts. This is in addition to other local subnets you may have that hosts your kubernetes cluster like the IPPool that you may configure for your k8s addressing:
    NO_PROXY URL's listed:

            github-releases.githubusercontent.com

            github.com

            app.terraform.io

            registry.terraform.io

            releases.hashicorp.com

            archivist.terraform.io

### Provision infrastructure and deploy App services with AppDynamics instrumentation


## Step 1: Importing ICO template for App Services Deployment and Instrumentation

Clone the following github repo to get the ICO template:

https://github.com/CiscoDevNet/IcoTemplates.git

Import the template ** ExportIksApp.json ** in Intersight:

![alt text](https://github.com/prathjan/images/blob/main/importiksapp.png?raw=true)

Review the worflows imported. The workflows leverage IST/TFCB Git repos referenced in Step 2 here: https://developer.cisco.com/codeexchange/github/repo/CiscoDevNet/AppDynamicsIKSIST

![alt text](https://github.com/prathjan/images/blob/main/summary4.png?raw=true)

In short, the following workflows are the ones referenced in the above main workflow:

AppdIksWorkspace - Sets up the TFCB workspaces for:

    Tfiks-Global - Sets up the global vars for all the workspaces

    Tfiks-host - Retrieves the k8s cluster master node IP to use for some utility functions

    Tfiks-Rbac - Runs the RBAC utility to setup user/role/license rules

    Tfiks-Metrics - Helm deployment of metrics server in k8s cluster

    Tfiks-ClusterA - Helm deployment of AppDynamics Cluster Agent

    Tfiks-AppSvc - Helm deployment of containerized Application Services

    Tfiks-Load - Runs JMeter based application load generator

    Tfiks-Remove - Removes all resources provisioned and removes the TFCB workspaces

UpdateIksVars - Updates the variables for the above TFCB workspaces

InvokeIksBasePlans - Sets up the base cluster dependencies before app is installed

InvokeIksAppPlans - Installs the Apps with AppDynamics auto instrumentation

## Step 2: Importing ICO template for App Services Load Generation

Import the template ** ExportIksLoad.json ** in Intersight:

![alt text](https://github.com/prathjan/images/blob/main/importiksload.png?raw=true)

Review the worflows imported:

![alt text](https://github.com/prathjan/images/blob/main/sumload.png?raw=true)

The above workflow runs the application load generator.

## Step 3: Importing ICO template for App Services Decommissioning

Import the template ** ExportIksDestroy.json ** in Intersight:

![alt text](https://github.com/prathjan/images/blob/main/importiksdestroy.png?raw=true)

Executing the imported workflow will delete resources created and also destroy the TFCB workspaces

## Step 4: Setup Tfiks-Global Variables

In Intersight, you will set up the data specific to your environment in this step. 

Open Orchestration-> UpdateIksVars->Add Global Variables Task:

![alt text](https://github.com/prathjan/images/blob/main/iksglobalvar1.png?raw=true)

Add the following variables, the data will be as it applies to your own specific environment:

appport - this is the application port that will be targetted by the load generator. For this use case, use 30080 since the nodePort for this App is provisioned as this

nbrapm - this is the number of app agent needed for this application. We have 8 services and so allocate 8 app agents here

nbrma - this is the number of machine agent licenses needed. For this use case, we have one clusteragent here so allocate 1

nbrsim - this license enables an add-on module to the machine agent that provides insights into the underlying infrastructure. For this use case, we have 1 clusteragent and so allocate 1 sim agent as well.

nbrnet(eg. 0), this is the .NET licenses needed. For this use case,set as 0

url - set to the url of the SAAS Controller. For this use case, we are leveraging the devnet SAAS controller and so set it to this: https://devnet.saas.appdynamics.com

namespaces - these are the k8s namespaces to be monitored by the AppDynamics cluster agent. Leave it as default

storename - your store name. eg. IKSChaiStore

username - For this use case, just enter a random str

dockeruser - your user name for containers.cisco.com


Next, Open Orchestration-> UpdateIksVars->Add Global Variables Sensitive Task:

![alt text](https://github.com/prathjan/images/blob/main/iksglobalvar3.png?raw=true)

Add the following variables, TBD's will be as it applies to your own specific environment:

password	- just enter a random str for now, not used for this use case

dockerpass - your password for containers.cisco.com

privatekey - base64 encoded private ssh key that you used when creating your IKS cluster

## Step 5: Setup Tfiks-Host Variables

Open Orchestration-> UpdateLegacyVars->Add Tfiks-Host Variables Task:

![alt text](https://github.com/prathjan/images/blob/main/ikshostvar.png?raw=true)

Add the following variables, TBD's will be as it applies to your own specific environment:

ikswsname - set this to be the workspace that you used to provision the IKS cluster, eg. intersight-tfb-iksgen

org - set this to the TFCB org, eg. Lab14

## Step 6: Setup Tfiks-Rbac Variables

Open Orchestration-> UpdateLegacyVars->Add Tfiks-Rbac Variables Task:

![alt text](https://github.com/prathjan/images/blob/main/iksrbac.png?raw=true)

Add the following variables, TBD's will be as it applies to your own specific environment:

ikswsname - set this to be the workspace that you used to p[rovision the IKS cluster, eg. intersight-tfb-iksgen

org - set this to the TFCB org, eg. Lab14

hostwsname - set this to the workspace of tfikshost, eg. Tfiks-Host

globalwsname - set this to the global workspace, eg. Tfiks-Global

## Step 7: Setup Tfiks-Metrics Variables

Open Orchestration-> UpdateLegacyVars->Add Tfiks-Metrics Variables Task:

![alt text](https://github.com/prathjan/images/blob/main/iksmetrics.png?raw=true)

Add the following variables, TBD's will be as it applies to your own specific environment:

org - set this to the TFCB org, eg. Lab14

hostwsname - set this to the workspace of tfikshost, eg. Tfiks-Host

globalwsname - set this to the global workspace, eg. Tfiks-Global

## Step 8: Setup Tfiks-ClusterA Variables

Open Orchestration-> UpdateLegacyVars->Add Tfiks-ClusterA Variables Task:

![alt text](https://github.com/prathjan/images/blob/main/ikscluster.png?raw=true)

Add the following variables, TBD's will be as it applies to your own specific environment:

org - set this to the TFCB org, eg. Lab14

hostwsname - set this to the workspace of tfikshost, eg. Tfiks-Host

globalwsname - set this to the global workspace, eg. Tfiks-Global

ikswsname - set this to be the workspace that you used to p[rovision the IKS cluster, eg. intersight-tfb-iksgen

## Step 9: Setup Tfiks-App Variables

Open Orchestration-> UpdateLegacyVars->Add Tfiks-App Variables Task:

![alt text](https://github.com/prathjan/images/blob/main/iksapp.png?raw=true)

Add the following variables, TBD's will be as it applies to your own specific environment:

ikswsname - set this to be the workspace that you used to p[rovision the IKS cluster, eg. intersight-tfb-iksgen

org - TBD (eg. Lab14)

## Step 10: Setup Tfiks-Load Variables

Open Orchestration-> UpdateLegacyVars->Add Tfiks-Load Variables Task:

![alt text](https://github.com/prathjan/images/blob/main/iksload.png?raw=true)

Add the following variables, TBD's will be as it applies to your own specific environment:

trigcount - some integer valuie, eg. 20

org - set this to the TFCB org, eg. Lab14

hostwsname - set this to the workspace of tfikshost, eg. Tfiks-Host

globalwsname - set this to the global workspace, eg. Tfiks-Global

## Step 11: Setup Tfiks-Remove Variables

Open Orchestration-> UpdateLegacyVars->Add Tfiks-Remove Variables Task:

![alt text](https://github.com/prathjan/images/blob/main/iksremove.png?raw=true)

Add the following variables, TBD's will be as it applies to your own specific environment:

org - set this to the TFCB org, eg. Lab14

hostwsname - set this to the workspace of tfikshost, eg. Tfiks-Host

globalwsname - set this to the global workspace, eg. Tfiks-Global

## Step 10: Execute ICO Workflow for App Services Deployment and Instrumentation

Open the AppdIksIco workflow and execute:

![alt text](https://github.com/prathjan/images/blob/main/exeiks.png?raw=true)

You will be prompted for the following data:

![alt text](https://github.com/prathjan/images/blob/main/exeiksparams.png?raw=true)

Pick up the Agent Pool ID and token from your TFCB account

![alt text](https://github.com/prathjan/images/blob/main/tfcb1.png?raw=true)

![alt text](https://github.com/prathjan/images/blob/main/tfcb2.png?raw=true)

## Step 11: Execute ICO Workflow for App Services Load Generation

Open the OnlyLoad workflow and execute:

![alt text](https://github.com/prathjan/images/blob/main/exeiksload.png?raw=true)

## Step 12 View AppDynamics Insights

### View Application Insights in AppDynamics 

Checkout the application insights in AppDynamics:

![alt text](https://github.com/prathjan/images/blob/main/iksappd2.png?raw=true)

![alt text](https://github.com/prathjan/images/blob/main/iksappd3.png?raw=true)

### View Application Insights in Intersight

Checkout the infrastructure insights in Intersight:

![alt text](https://github.com/prathjan/images/blob/main/iksappd4.png?raw=true)

### Undeploy applications and deprovision infrastructure

Open the ICO workflow AppdIksWorkspaceDelete imported earlier in Step 3 above. Execute to remove all the entities created in AppDynamics and TFCB. Select Organization, Target, CloudOrg, PoolID, Token as before and Execute: 

![alt text](https://github.com/prathjan/images/blob/main/iksdeswf.png?raw=true)

Due to a known error, you will have to manually delete the IKSChaiStore application from AppDynamics to complete the cleanup:

![alt text](https://github.com/prathjan/images/blob/main/iksdes.png?raw=true)
            

