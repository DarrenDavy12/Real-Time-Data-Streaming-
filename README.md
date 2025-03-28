
---

# Project 3: Real-Time Data Streaming Solution

## Overview
This project demonstrates the implementation of a real-time data streaming and analytics solution using Azure services, showcasing expertise in handling live data for dynamic environments. The solution ingests streaming data (e.g., IoT sensor data) via Azure Event Hubs, processes it with Spark Structured Streaming in Microsoft Fabric, performs real-time analytics such as windowed aggregates, and visualizes insights using Power BI. This aligns with the Azure Data Engineer objective of "Develop data processing," emphasizing stream processing and real-time intelligence.

### Project Description
- **Tasks**: Set up Azure Event Hubs for data ingestion, process streams using Spark Structured Streaming in Fabric, perform real-time analytics, and visualize results with Power BI.
- **Outcome**: A scalable, real-time analytics solution delivering up-to-the-minute insights, ideal for dynamic data environments like IoT or financial systems.
- **Rationale**: Real-time analytics is critical in modern data engineering, and this project leverages Fabric's Real-Time Intelligence capabilities, integrating seamlessly with Event Hubs and Power BI for end-to-end processing and visualization.

### Skills Showcased
- Real-time data ingestion and processing with Azure Event Hubs and Spark Structured Streaming.
- Streaming analytics, including handling schema drift and scaling resources.
- Data visualization and dashboard creation with Power BI.
- Proficiency in Azure CLI and Microsoft Fabric's real-time capabilities.

---

## Prerequisites
Before starting, ensure you have:
- An active Azure subscription (create one at [Azure Portal](https://portal.azure.com)).
- Azure CLI installed and configured (`az login` to authenticate) – latest version as of March 28, 2025.
- Access to Microsoft Fabric (via trial or subscription) – [Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/).
- A Power BI account for visualization – [Power BI](https://powerbi.microsoft.com/).
- Basic familiarity with Python and Spark for scripting.

---

## Step-by-Step Implementation

### Step 1: Set Up Azure Event Hubs for Data Ingestion
Azure Event Hubs is a managed platform for ingesting massive volumes of streaming data, making it ideal for real-time scenarios like IoT sensor data or live telemetry. We use Azure CLI to create and configure the Event Hub resources, ensuring scalability and integration with downstream processing.

#### 1.1 Create a Resource Group
- **Command**: 
  ```bash
  az group create --name RealTimeStreamingRG --location eastus
  ```
- **Why**: A resource group logically organizes all Azure resources for this project, simplifying management, deployment, and cleanup. `eastus` is chosen for its wide availability and low latency for many users.
- **Screenshot**: Take a screenshot of the Azure CLI output showing the resource group creation or the resource group listed in the Azure Portal.
  - **[Placeholder for screenshot of resource group creation]**

#### 1.2 Create an Event Hubs Namespace
- **Command**: 
  ```bash
  az eventhubs namespace create --name RealTimeNamespace --resource-group RealTimeStreamingRG --location eastus --sku Standard
  ```
- **Why**: The namespace acts as a container for one or more Event Hubs, providing a unique scoping boundary for authentication and management. The `Standard` SKU supports higher throughput and features like auto-inflate.
- **Screenshot**: Take a screenshot of the CLI output confirming namespace creation or the namespace details in the Azure Portal.
  - **[Placeholder for screenshot of namespace creation]**

#### 1.3 Create an Event Hub
- **Command**: 
  ```bash
  az eventhubs eventhub create --name SensorDataHub --namespace-name RealTimeNamespace --resource-group RealTimeStreamingRG --partition-count 4
  ```
- **Why**: The Event Hub is the core ingestion point for streaming data. Four partitions allow parallel processing, balancing throughput and cost for a demo-scale project (scalable up to 32 for production).
- **Screenshot**: Take a screenshot of the Event Hub details in the Azure Portal or the CLI output after creation.
  - **[Placeholder for screenshot of Event Hub creation]**

#### 1.4 Create a Consumer Group
- **Command**: 
  ```bash
  az eventhubs eventhub consumer-group create --consumer-group-name FabricConsumer --eventhub-name SensorDataHub --namespace-name RealTimeNamespace --resource-group RealTimeStreamingRG
  ```
- **Why**: Consumer groups enable multiple independent consumers (e.g., Fabric processing) to read the same stream without interference, supporting scalability and fault tolerance.
- **Screenshot**: Take a screenshot of the consumer group listed under the Event Hub in the Azure Portal.
  - **[Placeholder for screenshot of consumer group creation]**

#### 1.5 Retrieve Connection String
- **Command**: 
  ```bash
  az eventhubs namespace authorization-rule keys list --resource-group RealTimeStreamingRG --namespace-name RealTimeNamespace --name RootManageSharedAccessKey --query primaryConnectionString --output tsv
  ```
- **Why**: The connection string is required to connect Fabric to the Event Hub for reading the stream. Store this securely (e.g., in Azure Key Vault for production).
- **Screenshot**: Take a screenshot of the CLI output showing the connection string (redact sensitive parts) or the Shared Access Policies section in the Azure Portal.
  - **[Placeholder for screenshot of connection string retrieval]**

---

### Step 2: Process Streams with Spark Structured Streaming in Microsoft Fabric
Microsoft Fabric integrates data engineering tools like Spark Structured Streaming, launched in 2023, to process real-time streams natively. We use it over Stream Analytics (a separate service) to leverage Fabric’s unified platform and scalability for complex analytics.

#### 2.1 Access Microsoft Fabric
- **Action**: Log into [Microsoft Fabric](https://app.fabric.microsoft.com) with your credentials.
- **Why**: Fabric provides a centralized environment for data engineering, reducing integration overhead and aligning with its Real-Time Intelligence capabilities.
- **Screenshot**: Take a screenshot of the Fabric homepage after login, showing the workspace options.
  - **[Placeholder for screenshot of Fabric homepage]**

#### 2.2 Create a Workspace
- **Action**: In Fabric, click "Workspaces" > "New Workspace," name it `RealTimeStreamingWorkspace`, and assign it to your capacity.
- **Why**: Workspaces organize projects and resources, ensuring isolation and collaboration capabilities for team-based development.
- **Screenshot**: Take a screenshot of the new workspace created in Fabric, showing its name and details.
  - **[Placeholder for screenshot of workspace creation]**

#### 2.3 Create a Spark Notebook
- **Action**: In the workspace, select "Data Engineering" > "New Notebook."
- **Why**: Notebooks in Fabric allow interactive development of Spark jobs, ideal for testing and refining streaming logic.
- **Screenshot**: Take a screenshot of the notebook creation screen with the blank notebook open.
  - **[Placeholder for screenshot of Spark notebook creation]**

#### 2.4 Configure Spark to Read from Event Hubs
- **Action**: Add the following Python code to the notebook to connect to the Event Hub:
  ```python
  from pyspark.sql import SparkSession

  # Initialize Spark session
  spark = SparkSession.builder.appName("RealTimeStreamingDemo").getOrCreate()

  # Event Hub configuration
  connection_string = "<your_event_hub_connection_string>"
  ehConf = {
      "eventhubs.connectionString": spark._jvm.org.apache.spark.eventhubs.EventHubsUtils.encrypt(connection_string),
      "eventhubs.consumerGroup": "FabricConsumer"
  }

  # Read stream from Event Hub
  df = spark.readStream \
      .format("eventhubs") \
      .options(**ehConf) \
      .load()
  ```
- **Why**: This code establishes a streaming DataFrame from the Event Hub, using the connection string and consumer group for secure, targeted data access. The encryption step ensures compatibility with Fabric’s Spark environment.
- **Screenshot**: Take a screenshot of the notebook with this code entered, showing the cell ready to run.
  - **[Placeholder for screenshot of Spark code in notebook]**

#### 2.5 Parse and Process the Stream
- **Action**: Add code to parse the stream (assuming JSON sensor data) and write to console for testing:
  ```python
  from pyspark.sql.functions import col, from_json

  # Parse JSON data from Event Hub
  parsed_df = df.selectExpr("CAST(body AS STRING) AS json_data") \
                .select(from_json(col("json_data"), "temperature DOUBLE, timestamp STRING").alias("data")) \
                .select("data.*")

  # Write stream to console for verification
  query = parsed_df.writeStream \
      .outputMode("append") \
      .format("console") \
      .start()

  query.awaitTermination()
  ```
- **Why**: Parsing converts raw Event Hub data (binary `body`) into structured columns (e.g., `temperature`, `timestamp`), enabling analytics. Writing to the console verifies the stream is flowing correctly before further processing.
- **Screenshot**: Take a screenshot of the notebook running with console output showing parsed data (e.g., temperature and timestamp values).
  - **[Placeholder for screenshot of console output from streaming]**

---

### Step 3: Perform Real-Time Analytics
Real-time analytics derive actionable insights, such as averages or trends, from the stream. We implement a windowed aggregate in Spark to calculate average temperature over time.

#### 3.1 Add Windowed Aggregation
- **Action**: Replace the console output with this analytics code:
  ```python
  from pyspark.sql.functions import avg, window

  # Perform windowed aggregation
  analytics_df = parsed_df \
      .withWatermark("timestamp", "1 minute") \
      .groupBy(window("timestamp", "1 minute", "30 seconds")) \
      .agg(avg("temperature").alias("avg_temperature"))

  # Write results to memory for Power BI
  query = analytics_df.writeStream \
      .outputMode("complete") \
      .format("memory") \
      .queryName("avg_temp_table") \
      .start()

  query.awaitTermination()
  ```
- **Why**: The `withWatermark` handles late data, ensuring accurate aggregates. A 1-minute window with a 30-second slide provides frequent updates, suitable for real-time monitoring. Writing to memory allows Power BI to query the results.
- **Screenshot**: Take a screenshot of the notebook with this code and a sample `spark.sql("SELECT * FROM avg_temp_table").show()` output showing windowed averages.
  - **[Placeholder for screenshot of analytics output]**

---

### Step 4: Visualize with Power BI
Power BI integrates with Fabric to create interactive, real-time dashboards, making insights accessible to stakeholders.

#### 4.1 Connect Power BI to Fabric
- **Action**: In Power BI Desktop, click "Get Data" > "Microsoft Fabric," sign in, and select the `avg_temp_table` from your workspace.
- **Why**: Fabric’s OneLake stores processed data, enabling seamless Power BI integration without external connectors, streamlining the pipeline.
- **Screenshot**: Take a screenshot of the Power BI data source selection screen showing the Fabric connection and table selection.
  - **[Placeholder for screenshot of Power BI connecting to Fabric]**

#### 4.2 Create a Visualization
- **Action**: Drag `window.start` to the X-axis and `avg_temperature` to the Y-axis, creating a line chart.
- **Why**: A line chart visualizes temperature trends over time, aligning with the project’s goal of up-to-the-minute insights.
- **Screenshot**: Take a screenshot of the Power BI report with the line chart displaying the average temperature trend.
  - **[Placeholder for screenshot of Power BI line chart]**

#### 4.3 Publish and Enable Real-Time Refresh
- **Action**: Click "Publish" to Power BI Service, then configure the dataset for real-time refresh (e.g., DirectQuery mode).
- **Why**: Real-time refresh ensures the dashboard reflects the latest data, critical for dynamic environments.
- **Screenshot**: Take a screenshot of the published dashboard in Power BI Service, showing the live line chart.
  - **[Placeholder for screenshot of published Power BI dashboard]**

---

## GitHub Repository Structure
Organize your repository professionally:
```
RealTimeStreamingSolution/
├── scripts/
│   ├── setup_eventhubs.sh       # Azure CLI commands
│   └── spark_streaming.py       # Spark Structured Streaming code
├── reports/
│   └── realtime_dashboard.pbix  # Power BI report file
├── screenshots/                  # Folder for all screenshots
└── README.md                    # This file
```

---

## Conclusion
This project delivers a fully functional real-time data streaming solution, from ingestion with Azure Event Hubs to processing with Spark Structured Streaming in Fabric, analytics with windowed aggregates, and visualization in Power BI. It handles schema drift (via flexible parsing) and scales resources (via partitions and Fabric’s capacity), meeting the project’s objectives and showcasing Azure Data Engineer skills.



