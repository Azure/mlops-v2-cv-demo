# MLOps V2 Computer Vision Quickstart

This repository is a prebuilt project demonstrating a computer vision MLOps scenario using Azure Machine Learning and GitHub workflows.


This project was generated using the [MLOps v2 Solution Accelerator](https://github.com/Azure/mlops-v2) using the following project generation parameters:

| Parameter  | Value | 
| --- | --- | 
| Project Type: | cv |  
| Azure ML Interface: | aml-cli-v2 |  
| CI/CD Platform | github-actions |  
| IAC Provider | terraform |


The project is organized according to the table:

| Location | Contents | 
| --- | --- |
| `.github/workflows/` | GitHub workflows for infrastructure, training pipeline, and model deployment |
| `data/` | Sample request data to test the deployed endpoint |
|`data-science/` | Source code for model training |
| `images/` | Images for this README.md |
| `infrastructure/` | Terraform templates for Azure ML infrastructure |
| `mlops/azureml/` | Azure ML training and deployment pipelines |

</td></tr> </table>

Below is a quickstart to deploying this prebuilt project. Refer to the [MLOps v2 Solution Accelerator](https://github.com/Azure/mlops-v2) project for more comprehensive documentation and deployment guides for bootstrapping your own MLOps projects.

## Steps to Deploy

Clone this repository to your own GitHub organization and follow the steps below to deploy the demo.

1. [Create an Azure Service Principal and configure GitHub actions secrets.](#configure-github-actions-secrets)
2. [Configure dev and/or prod environments and create a dev branch.](#configure-azure-ml-environment-parameters)
3. [Use a GitHub workflow to create Azure ML infrastructure for dev and/or prod environments.](#deploy-azure-machine-learning-infrastructure)
4. [Use a GitHub workflow to create and run a pytorch vision model training pipeline in Azure ML.](#train-a-pytorch-classifier-in-azure-machine-learning)
5. [Use a GitHub workflow to deploy the vision model as a real-time endpoint in Azure ML.](#deploy-the-registered-model-to-an-online-endpoint)

---

## Configure GitHub Actions Secrets

   This step creates a service principal and GitHub secrets to allow the GitHub action workflows to create and interact with Azure Machine Learning Workspace resources.
   
   From the command line, execute the following Azure CLI command with your choice of a service principal name:
   
   > `# az ad sp create-for-rbac --name <service_principal_name> --role contributor --scopes /subscriptions/<subscription_id> --sdk-auth`
   
   You will get output similar to below:

   >`{`  
   > `"clientId": "<service principal client id>",`  
   > `"clientSecret": "<service principal client secret>",`  
   > `"subscriptionId": "<Azure subscription id>",`  
   > `"tenantId": "<Azure tenant id>",`  
   > `"activeDirectoryEndpointUrl": "https://login.microsoftonline.com",`  
   > `"resourceManagerEndpointUrl": "https://management.azure.com/",`  
   > `"activeDirectoryGraphResourceId": "https://graph.windows.net/",`  
   > `"sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",`  
   > `"galleryEndpointUrl": "https://gallery.azure.com/",`  
   > `"managementEndpointUrl": "https://management.core.windows.net/"`  
   > `}`
   
   Copy all of this output, braces included. From your GitHub project, select **Settings**:


<p align="left">
    <img src="./images/gh-settings.png" alt="GitHub Settings" width="50%" height="50%"/>
</p>

   Then select **Secrets**, then **Actions**:

 <p align="left">
    <img src="./images/gh-secrets.png" alt="GitHub Secrets" width="40%" height="40%"/>
</p>

Select **New repository secret**. Name this secret **AZURE_CREDENTIALS** and paste the service principal output as the content of the secret.  Select **Add secret**.

> **Note:**  
> As this infrastructure is deployed using terraform, add the following additional GitHub secrets using the corresponding values from the service principal output as the content of the secret:  
> 
> **ARM_CLIENT_ID**  
> **ARM_CLIENT_SECRET**  
> **ARM_SUBSCRIPTION_ID**  
> **ARM_TENANT_ID**  

The GitHub configuration is complete.

---

## Configure Azure ML Environment Parameters

   In your Github project repository, there are two configuration files in the root, `config-infra-dev.yml` and `config-infra-prod.yml`. These files are used to define and deploy Dev and Prod Azure Machine Learning environments. With the default deployment, `config-infra-prod.yml` will be used when working with the main branch or your project and `config-infra-dev.yml` will be used when working with any non-main branch.

   It is recommended to first create a dev branch from main and deploy this environment first.

   Edit each file to configure a namespace, postfix string, Azure location, and environment for deploying your Dev and Prod Azure ML environments. Default values and settings in the files are show below:

   > ```bash
   > namespace: mlopsv2 #maximum of 6 characters.  
   > postfix: 0001  
   > location: eastus  
   > environment: dev  
   > enable_aml_computecluster: true  
   > enable_monitoring: false  
   >```
   
   The first four values are used to create globally unique names for your Azure environment and contained resources. Edit these values to your liking then save, commit, push, or pr to update these files in the project repository. Leave `enable_monitoring` set to `false` for this demo. 

   As this is a deep learning workload, ensure your subscription and Azure location has available GPU compute. 
 
 ---

 ## Deploy Azure Machine Learning Infrastructure

   In your GitHub project repository, select **Actions**

   ![GH-actions](./images/gh-actions.png)

   This will display the pre-defined GitHub workflows associated with your project. For a classical machine learning project, the available workflows will look similar to this:

<p align="left">
    <img src="./images/gh-workflows.png" alt="centered image" width="50%" height="50%"/>
</p>


   Depending on the the use case, available workflows may vary. Select the workflow to 'deploy-infra'. In this scenario, the workflow to select would be **tf-gha-deploy-infra.yml**. This would deploy the Azure ML infrastructure using GitHub Actions and Terraform.

   <img src="./images/gh-deploy-infra.png" width="50%" height="50%">

   On the right side of the page, select **Run workflow** and select the branch to run the workflow on. This will deploy Dev Infrastructure if run on a dev branch or Prod infrastructure if running on main. Monitor the pipeline for successful completion.

   ![GH-infra-pipeline](./images/gh-infra-pipeline.png)

   When the pipeline has complete successfully, you can find your Azure ML Workspace and associated resources by logging in to the Azure Portal.

   Next, a sample model training pipeline will be deployed into the new Azure Machine Learning environment.

---

## Train a Pytorch Classifier in Azure Machine Learning

The solution accelerator includes code and data for a sample machine learning pipeline which trains a dog breed classifier on the [Stanford Dogs Dataset](http://vision.stanford.edu/aditya86/ImageNetDogs/main.html). 

The https://github.com/sdonohoo/mlops-cv-demo/blob/main/.github/workflows/deploy-model-training-pipeline.ymlGitHub workflow creates both cpu and gpu-based compute clusters in the Azure ML workspace. The cpu cluster is used for the data registration job that downloads and registers the training image dataset in Azure ML. The gpu cluster is used by an Azure ML pipeline with a single component that trains and registers a pytorch classifier model on this dataset.

To deploy the model training pipeline in the previously created Azure ML workspace, select **Actions** in your GitHub project repository. 

   ![GH-actions](./images/gh-actions.png)

Then select the `deploy-cv-model-training-pipeline`.

<p align="left">
    <img src="./images/gh-workflows-train.png" alt="centered image" width="50%" height="50%"/>
</p>

As before, select  **Run workflow** on the right and select the branch to run from. This will run the workflow, create compute clusters, register the dataset, and deploy the training pipeline in Azure ML.

![GH-actions](./images/gh-cv-model-training-workflow.png)

Once the run-model-training-pipeline job begins running, you can follow the execution of this job in the Azure ML workspace. 

<p align="center">
    <img src="./images/azureml-model-training.png" alt="centered image" width="50%" height="50%"/>
</p>

When the Azure ML pipeline completes, the trained model should be registered in the workspace.

<p align="left">
    <img src="./images/azureml-registered-models.png" alt="centered image" width="50%" height="50%"/>
</p>

Next, the registered model will be deployed to a real-time endpoint for classifying new images.

---

## Deploy the Registered Model to an Online Endpoint

This step uses a GitHub workflow to deploy the registered model to an Azure ML Managed Online Endpoint for predicting the class of new images.

This workflow will register the interference environment with prerequisite python packages, create an Azure ML endpoint, create a deployment of the registered model to that endpoint, then allocate traffic to the endpoint.

To run the workflow to deploy the registered model as a managed online endpoint, select **Actions** in your GitHub project repository.

   ![GH-actions](./images/gh-actions.png)

Then select the `deploy-online-endpoint-pipeline`.

<p align="left">
    <img src="./images/gh-deploy-online-endpoint.png" alt="centered image" width="50%" height="50%"/>
</p>

Run the workflow.

![GH-actions](./images/gh-workflow-deploy-endpoint.png)

As the create-endpoint job begins, you can monitor the ednpoint creation, deployment creation, and traffic allocation in the Azure ML workspace.

<p align="left">
    <img src="./images/azureml-online-endpoint.png" alt="centered image" width="50%" height="50%"/>
</p>

---

## Next Steps

* Explore the construction of the GitHub workflows to deploy infastructure and deploy pipelines and register artifacts in Azure ML.
* Explore the python code in `data-science/` and Azure ML pipeline definitions in `mlops/azureml/` to understand how to adapt your own computer vision code to this pattern.
* Configure the GitHub repository and workflows to fit your MLOps workflows and policies utilizing prod/dev branches, branch protection, and pull requests.


## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
