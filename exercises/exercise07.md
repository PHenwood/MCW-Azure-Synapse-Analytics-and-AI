## Exercise 7: Machine Learning

[< Previous Exercise](../exercises/exercise06.md#exercise-6-security) - **[Home](https://github.com/tayganr/MCW-Azure-Synapse-Analytics-and-AI#azure-synapse-analytics-and-ai-hands-on-lab)** - [Next Exercise >](../exercises/exercise08.md#exercise-8-monitoring)

**Duration**: 60 minutes

**Contents**
* [Task 1: Create a SQL Datastore and source Dataset](#task-1-create-a-sql-datastore-and-source-dataset)
* [Task 2: Create compute infrastructure](#task-2-create-compute-infrastructure)
* [Task 3: Use a notebook in AML Studio to prepare data and create a Product Seasonality Classifier model using XGBoost](#task-3-use-a-notebook-in-aml-studio-to-prepare-data-and-create-a-product-seasonality-classifier-model-using-xgboost)
* [Task 4: Leverage Automated ML to create and deploy a Product Seasonality Classifier model](#task-4-leverage-automated-ml-to-create-and-deploy-a-product-seasonality-classifier-model)

Using Azure Synapse Analytics, data scientists are no longer required to use separate tooling to create and deploy machine learning models.

In this exercise, you will create multiple machine learning models. You will learn how to consume these models in your notebook. You will also deploy a model as a web service to Azure Container Instances and consume the service.

<div align="right"><a href="#exercise-7-machine-learning">↥ back to top</a></div>

### Task 1: Create a SQL Datastore and source Dataset

1. Open the lab resource group, locate and open the **amlworkspace{{suffix}}** Machine Learning resource.

    ![The lab resource group is shown with the Machine Learning resource selected](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/resourcelist_amlstudio.png "Machine learning resource")

2. On the **Overview** screen of the Machine Learning resource, select the **Studio web URL** link.

    ![The machine learning resource overview screen is selected with the Studio web URL link highlighted.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/machinelearning_overview.png "Machine learning overview screen")

3. From the left menu of **Azure Machine Learning Studio**, select the **Datastores** item.

    ![The Machine Learning Studio menu is shown with the Datastores item highlighted](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/amlstudio_datastores_menu.png "AML Studio menu")

4. On the **Datastores** screen top menu, select **+ New datastore**.

5. On the **New datastore** blade, configure it as follows and select **Create**:

    | Field | Value |
    |--------------|---------------|
    | New datastore (name) | sqlpool01 |
    | Datastore type | Azure SQL database |
    | Account selection method | From Azure subscription |
    | Subscription ID | Select the lab subscription. |
    | Server name / database name  | Select asaworkspace{{suffix}}/SQLPool01. |
    | Authentication type | SQL authentication |
    | User ID | asa.sql.admin |
    | Password | The SQL Admin password you chose when deploying the lab resources. |

    ![The new datastore blade is shown populated with the preceding values.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/amlstudio_sqlpooldatasource.png "New datastore blade")

6. From the left menu, select **Datasets**, and with the **Registered datasets** tab selected, expand the **+ Create dataset** button and select **From datastore**.

    ![The Datasets screen is displayed with the +Create dataset button highlighted.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/createfirstamldataset.png "AML Studio Datasets screen")

7. In the **Create dataset from datastore** Basic info form, name the dataset **AggregatedProductSeasonality** and select **Next**.

    ![The basic info form is displayed populated with the preceding values.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/createdataset_basicinfo.png "Dataset basic info form")

8. On the **Datastore selection** form, select **Previously created datasource**, choose **sqlpool01** from the list and select the **Select datastore** button.

    ![The Datastore selection form is displayed as described above.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/amldatasetselectdatasource.png "The Datastore selection form")

9. In the next **Datastore selection** form, enter the following **SQL query**. Then expand the **Advanced settings** and enter **100** for the **Query timeout (seconds)** value. Select **Next**:

    ```sql
    SELECT P.ProductId,P.Seasonality,S.TransactionDateId,COUNT(*) as TransactionItemsCount
    FROM wwi_mcw.SaleSmall S
    JOIN wwi_mcw.Product P ON S.ProductId = P.ProductId
    where TransactionDateId between 20190101 and 20191231
    GROUP BY P.ProductId ,P.Seasonality,S.TransactionDateId
    ```

    ![The datastore selection form is displayed populated with the preceding query.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_dataset_datastoreselectionquery.png "Dataset query details")

10. The **Settings and preview** data table will be displayed after a few moments. Review this data, then select the **Next** button.

    ![The settings and preview screen is displayed showing a table of data.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/amlstudio_dataset_settingsandpreview.png "The Settings and Preview screen")

11. Review the **Schema** field listing, then select **Next**.

    ![The Schema screen is displayed showing a listing of columns and their types.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/amlstudio_dataset_schema.png "The dataset Schema field listing")

12. On the **Confirm details** screen, select **Create**.

    ![The dataset Confirm details screen is displayed showing a summary of the choices from the previous steps.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_dataset_confirmdetails.png "The dataset Confirm details screen")

<div align="right"><a href="#exercise-7-machine-learning">↥ back to top</a></div>

### Task 2: Create compute infrastructure

1. From the left menu of Machine Learning Studio, select **Compute**.

2. On the **Compute** screen with the **Compute instances** tab selected. Choose the **Create** button.

    ![The Azure Machine Learning Studio compute screen is displayed, with the compute instances tab selected, and the Create button highlighted.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_createcomputebutton.png "Azure Machine Learning Compute screen")

3. On the **Create compute instance**, **Select virtual machine** form, configure it as follows, then select **Next**:

    | Field | Value |
    |--------------|---------------|
    | Virtual machine type | CPU |
    | Virtual machine size | Search for and select Standard_DS3_v2. |

    ![The new compute instance virtual machine form is displayed populated with the preceding values.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_newcomputeform.png "The new compute instance virtual machine form")

4. On the **Configure Settings** form, enter a globally unique **Compute name** of your choice, and select **Create**.

    ![The new compute instance settings form is displayed populated with a compute name](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_newcomputeform_settings.png "The new compute instance settings form")

5. Select the **Compute clusters** tab, and select **Create**.

6. On the **New compute cluster**, **Select virtual machine** form, configure the virtual machine as follows, then select **Next**:

    | Field | Value |
    |--------------|---------------|
    | Virtual machine priority | Dedicated |
    | Virtual machine type | CPU |
    | Virtual machine size | Search for and select Standard_DS3_v2. |

    ![The New compute cluster virtual machine form is displayed with the preceding values.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_cluster_settings.png "The New compute cluster virtual machine form")

7. On the **Configure Settings** form, configure it as follows, then select **Create**:

    | Field | Value |
    |--------------|---------------|
    | Compute name | automlcluster |
    | Minimum number of nodes | 0 |
    | Maximum number of nodes | 3 |
    | Idle seconds before scale down | 120 |

    ![The new compute cluster configure settings form is displayed populated with the preceding values.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_cluster_configsettings.png "The new compute cluster Configure settings form")

<div align="right"><a href="#exercise-7-machine-learning">↥ back to top</a></div>

### Task 3: Use a notebook in AML Studio to prepare data and create a Product Seasonality Classifier model using XGBoost

1. In Azure Machine Learning (AML) Studio, select **Notebooks** from the left menu.

2. In the **Notebooks** pane, select the **Upload** icon from the toolbar.

    ![In Azure Machine Learning Studio, the Notebooks item is selected from the left menu, and the Upload Icon is highlighted in the Notebooks panel.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_uploadnotebook_menu.png "Upload notebook")

3. In the **Open** dialog, select **Hands-on lab/artifacts/ProductSeasonality_sklearn.ipynb**. When prompted, check the boxes to **Overwrite if already exists** and **I trust contents of this file** and select **Upload**.

    ![A dialog is displayed with the Overwrite if already exists and the I trust contents of this file checkboxes checked.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_notebook_uploadwarning.png "File upload warning dialog")

4. In the top toolbar of the notebook, expand the **Editors** item, and select **Edit in Jupyter**.

    ![On the notebook toolbar, the Editors item is expanded with the Edit in Jupyter item selected.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_notebook_editinjupyter.png "Edit in Jupyter")

5. Review and run each cell in the notebook individually to gain understanding of the functionality being demonstrated.

>**Note**: Running this notebook in its entirety is required for the next task.

<div align="right"><a href="#exercise-7-machine-learning">↥ back to top</a></div>

### Task 4: Leverage Automated ML to create and deploy a Product Seasonality Classifier model

1. In Azure Machine Learning (AML) Studio, select **Experiments** from the left menu, then expand the **+ Create** button, and select **Automated ML run**.

    ![The AML Studio Experiments screen is shown with the Create button expanded and the Automated ML run item selected.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_experiment_create.png "The AML Studio Experiments screen")

2. In the previous task, we registered our PCA dataframe (named **pcadata**) to use with Auto ML. Select **pcadata** from the list and select **Next**.

    ![On the Select dataset screen, the pcadata item is selected from the dataset list.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_automl_datasetselection.png "The select dataset form is displayed")

3. On the **Configure run** screen, select the **Create a new compute** link beneath the **Select compute cluster** field.

4. Back on the **Configure run** form, name the experiment **ProductSeasonalityClassifier**, select **Seasonality** as the **Target column** and select **automlcluster** as the compute cluster. Select **Next**.

    ![The Configure run form is displayed populated with the preceding values.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/automl_experiment_configurerun.png "The Configure run form")

5. On the **Select task type** screen, select **Classification**, then choose **Finish**.

    ![The Select task type screen is displayed with the Classification item selected.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_automlrun_tasktypeform.png "The Select task type screen")

6. The experiment will then be run. It will take approximately 20-25 minutes for it to complete. Once it has completed, it will display the run results details. In the **Best model summary** box, select the **Algorithm name** link.

    ![The Run is shown as completed and the link below Algorithm name in the Best model summary box is selected.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_automl_run_bestmodel_details.png "Completed AutoML run details")

7. On the Model run screen, select **Deploy** from the top toolbar.

    ![The specific model run screen is shown with the Deploy button selected from the top toolbar.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_automl_deploybestmodel.png "The best model run")

8. On the **Deploy a model** blade, configure the deployment as follows, then select **Deploy**:

    | Field | Value |
    |--------------|---------------|
    | Name | productseasonalityclassifier |
    | Description | Product Seasonality Classifier. |
    | Compute type | Azure Container Instance |
    | Enable authentication | Off |

    ![The Deploy a model blade is shown populated with the preceding values.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_automl_deploymodelaci.png "The Deploy a model blade")

9. Once deployed, the Model summary will be displayed. You can view the endpoint by selecting the **Deploy status** link.

    ![The successful model deployment was successful and the Deploy status link is highlighted.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_automl_modeldeploysuccess.png "The Model summary screen")

10. Review the details of the deployed model service endpoint.

    ![The service endpoint details screen is displayed.](https://raw.githubusercontent.com/microsoft/MCW-Azure-Synapse-Analytics-and-AI/master/Hands-on%20lab/media/aml_automl_modelserviceendpointdetails.png "The service endpoint details screen")

<div align="right"><a href="#exercise-7-machine-learning">↥ back to top</a></div>
