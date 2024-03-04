# Trabalhando com Machine Learning na Prática no Azure ML



Explore Automated Machine Learning in Azure ML
==============================================

In this exercise, you’ll use the automated machine learning feature in Azure Machine Learning to train and evaluate a machine learning model. You’ll then deploy and test the trained model.

This exercise should take approximately **30** minutes to complete.

> **Note** To complete this lab, you will need an [Azure subscription](https://azure.microsoft.com/free) in which you have administrative access.

Create an Azure Machine Learning workspace
------------------------------------------

To use Azure Machine Learning, you need to provision an Azure Machine Learning workspace in your Azure subscription. Then you’ll be able to use Azure Machine Learning studio to work with the resources in your workspace.

> **Tip**: If you already have an Azure Machine Learning workspace, you can use that and skip to the next task.

1.  Sign into the [Azure portal](https://portal.azure.com) at `https://portal.azure.com` using your Microsoft credentials.
    
2.  Select **\+ Create a resource**, search for _Machine Learning_, and create a new **Azure Machine Learning** resource with the following settings:
    *   **Subscription**: _Your Azure subscription_.
    *   **Resource group**: _Create or select a resource group_.
    *   **Name**: _Enter a unique name for your workspace_.
    *   **Region**: _Select the closest geographical region_.
    *   **Storage account**: _Note the default new storage account that will be created for your workspace_.
    *   **Key vault**: _Note the default new key vault that will be created for your workspace_.
    *   **Application insights**: _Note the default new application insights resource that will be created for your workspace_.
    *   **Container registry**: None (_one will be created automatically the first time you deploy a model to a container_).
3.  Select **Review + create**, then select **Create**. Wait for your workspace to be created (it can take a few minutes), and then go to the deployed resource.
    
4.  Select **Launch studio** (or open a new browser tab and navigate to [https://ml.azure.com](https://ml.azure.com?azure-portal=true), and sign into Azure Machine Learning studio using your Microsoft account). Close any messages that are displayed.
    
5.  In Azure Machine Learning studio, you should see your newly created workspace. If not, select **All workspaces** in the left-hand menu and then select the workspace you just created.

Enable preview features
-----------------------

Some features of Azure Machine Learning are in preview, and need to be explicitly enabled in your workspace.

1.  In Azure Machine Learning Studio, click on **manage preview features** (the loud speaker icon - 🕫).   
  
    
2.  Enable the following preview feature:
    
    *   _Guided experience for submitting training jobs with serverless compute_

Use automated machine learning to train a model
-----------------------------------------------

Automated machine learning enables you to try multiple algorithms and parameters to train multiple models, and identify the best one for your data. In this exercise, you’ll use a dataset of historical bicycle rental details to train a model that predicts the number of bicycle rentals that should be expected on a given day, based on seasonal and meteorological features.

> **Citation**: _The data used in this exercise is derived from [Capital Bikeshare](https://www.capitalbikeshare.com/system-data) and is used in accordance with the published data [license agreement](https://www.capitalbikeshare.com/data-license-agreement)_.

1.  In [Azure Machine Learning studio](https://ml.azure.com?azure-portal=true), view the **Automated ML** page (under **Authoring**).
    
2.  Create a new Automated ML job with the following settings, using **Next** as required to progress through the user interface:
    
    **Basic settings**:
    
    *   **Job name**: mslearn-bike-automl
    *   **New experiment name**: mslearn-bike-rental
    *   **Description**: Automated machine learning for bike rental prediction
    *   **Tags**: _none_
    
    **Task type & data**:
    
    *   **Select task type**: Regression
    *   **Select dataset**: Create a new dataset with the following settings:
        
        *   **Data type**:
            *   **Name**: bike-rentals
            *   **Description**: Historic bike rental data
            *   **Type**: Tabular
        *   **Data source**:
            *   Select **From web files**
        *   **Web URL**:
            *   **Web URL**: `https://aka.ms/bike-rentals`
            *   **Skip data validation**: _do not select_
        *   **Settings**:
            *   **File format**: Delimited
            *   **Delimiter**: Comma
            *   **Encoding**: UTF-8
            *   **Column headers**: Only first file has headers
            *   **Skip rows**: None
            *   **Dataset contains multi-line data**: _do not select_
        *   **Schema**:
            *   Include all columns other than **Path**
            *   Review the automatically detected types
        
        Select the **bike-rentals** dataset after you’ve created it.
        
    
    **Task settings**:
    
    *   **Task type**: Regression
    *   **Dataset**: bike-rentals
    *   **Target column**: Rentals (integer)
    *   **Additional configuration settings**:
        *   **Primary metric**: Normalized root mean squared error
        *   **Explain best model**: _Unselected_
        *   **Use all supported models**: Unselected. _You’ll restrict the job to try only a few specific algorithms._
        *   **Allowed models**: _Select only **RandomForest** and **LightGBM** — normally you’d want to try as many as possible, but each model added increases the time it takes to run the job._
    *   **Limits**: _Expand this section_
        *   **Max trials**: 3
        *   **Max concurrent trials**: 3
        *   **Max nodes**: 3
        *   **Metric score threshold**: 0.85 (_so that if a model achieves a normalized root mean squared error metric score of 0.085 or less, the job ends._)
        *   **Timeout**: 15
        *   **Iteration timeout**: 5
        *   **Enable early termination**: _Selected_
    *   **Validation and test**:
        *   **Validation type**: Train-validation split
        *   **Percentage of validation data**: 10
        *   **Test dataset**: None
    
    **Compute**:
    
    *   **Select compute type**: Serverless
    *   **Virtual machine type**: CPU
    *   **Virtual machine tier**: Dedicated
    *   **Virtual machine size**: Standard\_DS3\_V2
    *   **Number of instances**: 1
3.  Submit the training job. It starts automatically.
    
4.  Wait for the job to finish. It might take a while — now might be a good time for a coffee break!
    

Review the best model
---------------------

When the automated machine learning job has completed, you can review the best model it trained.

1.  On the **Overview** tab of the automated machine learning job, note the best model summary. 
    
    > **Note** You may see a message under the status “Warning: User specified exit score reached…”. This is an expected message. Please continue to the next step.
    
2.  Select the text under **Algorithm name** for the best model to view its details.
    
3.  Select the **Metrics** tab and select the **residuals** and **predicted\_true** charts if they are not already selected.
    
    Review the charts which show the performance of the model. The **residuals** chart shows the _residuals_ (the differences between predicted and actual values) as a histogram. The **predicted\_true** chart compares the predicted values against the true values.
    

Deploy and test the model
-------------------------

1.  On the **Model** tab for the best model trained by your automated machine learning job, select **Deploy** and use the **Web service** option to deploy the model with the following settings:
    *   **Name**: predict-rentals
    *   **Description**: Predict cycle rentals
    *   **Compute type**: Azure Container Instance
    *   **Enable authentication**: _Selected_
2.  Wait for the deployment to start - this may take a few seconds. The **Deploy status** for the **predict-rentals** endpoint will be indicated in the main part of the page as _Running_.
3.  Wait for the **Deploy status** to change to _Succeeded_. This may take 5-10 minutes.

Test the deployed service
-------------------------

Now you can test your deployed service.

1.  In Azure Machine Learning studio, on the left hand menu, select **Endpoints** and open the **predict-rentals** real-time endpoint.
    
2.  On the **predict-rentals** real-time endpoint page view the **Test** tab.
    
3.  In the **Input data to test endpoint** pane, replace the template JSON with the following input data:
    
         {
           "Inputs": { 
             "data": [
               {
                 "day": 1,
                 "mnth": 1,   
                 "year": 2022,
                 "season": 2,
                 "holiday": 0,
                 "weekday": 1,
                 "workingday": 1,
                 "weathersit": 2, 
                 "temp": 0.3, 
                 "atemp": 0.3,
                 "hum": 0.3,
                 "windspeed": 0.3 
               }
             ]    
           },   
           "GlobalParameters": 1.0
         }
        
    
4.  Click the **Test** button.
    
5.  Review the test results, which include a predicted number of rentals based on the input features - similar to this:
    
         {
           "Results": [
             444.27799000000000
           ]
         }
        
    
    The test pane took the input data and used the model you trained to return the predicted number of rentals.
    

Let’s review what you have done. You used a dataset of historical bicycle rental data to train a model. The model predicts the number of bicycle rentals expected on a given day, based on seasonal and meteorological _features_.

Clean-up
--------

The web service you created is hosted in an _Azure Container Instance_. If you don’t intend to experiment with it further, you should delete the endpoint to avoid accruing unnecessary Azure usage.

1.  In [Azure Machine Learning studio](https://ml.azure.com?azure-portal=true), on the **Endpoints** tab, select the **predict-rentals** endpoint. Then select **Delete** and confirm that you want to delete the endpoint.
    
    Deleting your compute ensures your subscription won’t be charged for compute resources. You will however be charged a small amount for data storage as long as the Azure Machine Learning workspace exists in your subscription. If you have finished exploring Azure Machine Learning, you can delete the Azure Machine Learning workspace and associated resources.
    

To delete your workspace:

1.  In the [Azure portal](https://portal.azure.com?azure-portal=true), in the **Resource groups** page, open the resource group you specified when creating your Azure Machine Learning workspace.
2.  Click **Delete resource group**, type the resource group name to confirm you want to delete it, and select **Delete**.

[MicrosoftLearning/AI-900-AIFundamentals](https://github.com/MicrosoftLearning/AI-900-AIFundamentals)
