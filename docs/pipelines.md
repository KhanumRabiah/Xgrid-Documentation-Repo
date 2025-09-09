## **Documenting Azure Data Factory Pipeline Dependencies**

This guide outlines the standard procedure for inspecting an Azure Data Factory (ADF) pipeline to extract and log its dependencies. The goal is to gather detailed information about data sources, destinations, and configurations for a dependency mapping sheet.

Note: Add images for better clarity

### **Prerequisites**

Before you begin, ensure you have the following:
* Access to the Microsoft Azure Portal.
* Permissions to activate the **Reader** role for the production environment via Privileged Identity Management (PIM).

---

### **Step 1: Activate Permissions**

Your first step is to ensure you have the necessary viewing rights for the production environment.

1.  Navigate to the Azure PIM portal using the following link to activate your role:
    * [PIM Activation Link](https://portal.azure.com/#view/Microsoft_Azure_PIMCommon/ActivationMenuBlade/~/azurerbac)
2.  Activate the **Reader** role as required. This will grant you read-only access to the necessary resources.

---

### **Step 2: Navigate to the Data Factory**

Once your permissions are active, locate the specific Data Factory instance you need to inspect.

1.  From the Azure Portal home page, navigate to **Data Factories**. You can use this direct link:
    * [Data Factories](https://portal.azure.com/#browse/Microsoft.DataFactory%2FdataFactories)
2.  Select the relevant Data Factory from the list.
3.  On the factory's overview page, click **Launch Studio**.
4.  If a pop-up window appears prompting you to log in to GitHub, you can close or skip it.



---

### **Step 3: Locate and Inspect the Pipeline**

Now you will find and analyze the specific pipeline to document its components.

1.  In the ADF Studio, navigate to the **Author** tab (pencil icon ✏️) from the left-hand menu.
2.  Use the search bar under the "Pipelines" section to find and click on the pipeline you are documenting.
3.  The pipeline will open on a canvas, displaying all its activities visually. Click on an individual activity (e.g., a "Copy data" or "Notebook" activity) to inspect its configuration details in the panel at the bottom of the screen.

---

### **Step 4: Gather Information for the Dependency Map**

For each relevant activity in the pipeline, carefully collect the following information. The location of this data depends on the activity type.

* **For data movement activities** (e.g., *Copy data*):
    * Select the **Source** tab to find the source dataset, path, and storage system.
    * Select the **Sink** tab to find the destination dataset, path, and storage system.

* **For compute activities** (e.g., *Databricks Notebook*):
    * Select the **Settings** tab. Here you can find the linked notebook path. You may need to open the notebook itself to identify specific secrets, sources, and sinks defined in the code.

* **To view connection details**:
    * The **Linked Service** for any source, sink, or compute cluster is specified within its respective configuration tab. For detailed information about a Linked Service, navigate to the **Manage** tab (toolbox icon 🧰) and click on **Linked services**.

### **Checklist of Information to Collect**

As you inspect each pipeline, record the following details in your dependency mapping sheet:

**Data Flow**
* **Source Datasets**: The name of the source dataset used in the activity.
* **Source Dataset Path (Schema)**: The specific file path, folder, or table (e.g., `container/folder/file.csv`).
* **Destination Datasets**: The name of the destination dataset.
* **Destination Dataset Path (Schema)**: The specific destination path or table.
* **Transport**: The method of data movement (e.g., Copy Activity, Data Flow).

**Infrastructure & Services**
* **Source Storage System**: The underlying technology for the source (e.g., Azure Blob Storage, Azure SQL Database).
* **Destination Storage System**: The underlying technology for the destination.
* **Linked Services**: The names of all linked services used to connect to data stores and compute resources.

**Security & Configuration**
* **Notebook Level Secrets**: Any secrets or credentials referenced within the notebook's code.
* **Cluster Level Secrets**: Secrets configured at the compute cluster level.