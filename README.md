# Real-Time Weather Reporting and Alert System (MS Azure & Fabric)

## TL;DR
This project implements a real-time weather monitoring system that fetches weather data (real-time weather conditions, air quality, alerts, forecast and so on) via a weather API, streams the data using Azure Event Hub, and processes it through Microsoft Fabric (Kusto DB) for real-time analysis, and reporting with Power BI. It also includes real-time email alerts for severe weather conditions. The solution explores both Azure Functions and Azure Databricks for data ingestion and highlights their cost and performance trade-offs.

## Objective
- **Real-time Weather Data:** Continuously ingest weather data (current weather, forecasts, alerts, and air quality) from a weather API.
- **Data Streaming & Processing:** Stream the data to Event Hubs and load it into Microsoft Fabric (Kusto DB) for further analysis.
- **Alerting & Reporting:** Use Power BI for data visualization and Data Activator for real-time email alerts on harsh weather conditions.
- **Cost & Performance Optimization:** Evaluate Azure Functions versus Azure Databricks to determine the most cost-effective and performant solution for this lightweight ingestion task.
- **Security & Management:** Secure sensitive keys using Azure Key Vault and manage access via role-based access control (RBAC).

## Architecture

![Architecture Diagram](Images/Architecture-diagram.gif)
The solution architecture is built on two main technology stacks:

### Azure Components

### Data Ingestion
- **Azure Functions:**  
  Utilized for serverless, cost-effective data ingestion. A timer-triggered function runs every 30 seconds to fetch data from the weather API and send it to EvenetHub to be finally loaded into KQL DB. Managed identities grant direct access to Event Stream and Key Vault for secure secret management.

- **Azure Databricks:**  
  Explored as an alternative for data ingestion. A Python script within a Databricks notebook fetches weather data and sends it to EvenetHub to be finally loaded into KQL DB. Secrets are managed using Azure Key Vault integrated with Databricks. However, for this specific use case, Azure Functions was preferred due to cost and performance considerations. PySpark commands are used to handle streaming and continuous data processing.

### Data Transport
- **Azure Event Hubs:**  
  Acts as the initial entry point for weather data events from Azure Functions or Databricks before routing to Event Stream. A namespace contains the event hub instance. Shared Access Policies manage authentication, with connection strings stored securely in Key Vault.

### Security
- **Azure Key Vault:**  
  Manages sensitive information such as API keys and connection strings. Roles like Key Vault Secrets User are assigned to control access.

- **Role-Based Access Control (RBAC):**  
  Implemented across Azure services to ensure secure and appropriate access. For instance, Azure Event Hubs Data Sender role is assigned to allow sending data to Event Hubs.

### Microsoft Fabric Components

#### Real-time Intelligence
- **Event Stream:**
  Part of Microsoft Fabric's real-time analytics capabilities, handling the streaming data flow from ingestion sources.

- **Event House (Kusto DB):**  
  A cloud-native database optimized for time-series analytics and real-time intelligence. Stores and processes the weather data stream for efficient querying and analysis.

#### Analytics and Automation
- **Power BI:**  
  Connects directly to Event House to provide interactive dashboards and reports, enabling real-time visualization of weather patterns and trends.

- **Data Activator:**  
  Monitors data streams in real-time and triggers automated actions. Configured to send email notifications through MS Outlook when severe weather conditions are detected.
 To see an example of email alerts sent by the system, check the [alert email sample](Images/alert-email.png).
 
## Cost and Performance Considerations

### Azure Services
- **Azure Functions vs Azure Databricks:**
  - Functions offers a serverless architecture with automatic scaling, making it cost-effective for periodic tasks like this. The consumption plan provides a free grant of executions, with minimal costs beyond that.
  - Databricks, while powerful for large-scale data processing, incurs higher costs due to the need for maintaining clusters. For simple tasks like API calls and data forwarding, Databricks may be less cost-effective.
  - For this project, Azure Functions was chosen for data ingestion due to its cost efficiency and simplicity. Monthly cost calculations for ingesting data every 30 seconds can be simulated using the [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/).
 - Note: The report dashboard updates have been temporarily suspended as the Fabric capacity was stopped to control costs, which could exceed $500 per month. The dashboard will resume updating when Fabric capacity and other instances are reactivated. For detailed cost breakdown, see [service costs](Images/cost.png).
 
### **Azure Functions Details**

| **Feature**          | **Details**                             |
|----------------------|-----------------------------------------|
| **Region**           | Canada Central                          |
| **Tier**             | Consumption                             |
| **Memory Size**      | 128 MB                                  |
| **Executions**       | 100                                     |
| **Execution Time**   | 0 ms                                    |
| **Monthly Cost**     | $0 (First 1,000,000 executions & 400,000 GB/s are free) <br> $0.20 per million executions after the free grant |
| **Plan**             | Free Grant Per Month                          |
| **Note**             | A storage account is created by default with each Functions app. The storage account is not included in the free grant. Standard storage rates and networking rates are charged separately as applicable. |

---

### **Azure Databricks Details**

| **Feature**          | **Details**                             |
|----------------------|-----------------------------------------|
| **Region**           | Canada Central                          |
| **Workload**         | All-Purpose Compute                     |
| **Category**         | All                                     |
| **Instance Series**  | All                                     |
| **Instance Type**    | P D3 v2 (4 vCPUs, 14 GB RAM, 200 GB Temp Storage, 0.75 Databricks Units) |
| **Virtual Machines Tier** | Premium                             |
| **Usage Hours**      | 730 Hours (Monthly)                     |
| **Monthly Cost**     | $546.41                                 |
| **Plan**             | Pay-as-you-go                           |

---

## Additional Details

### Weather API (weatherapi.com)
- **API Key:** Required for authentication. The key is stored securely in Azure Key Vault and accessed via managed identities.
- **API Endpoints:** The API provides various endpoints for current weather, forecasts, alerts, and air quality data.
- Supports up to 1 million requests per month for free which means we can hit their endpoints every 3 seconds for the whole month.
- Documentation available at [Weather API Docs](https://www.weatherapi.com/docs/).

### Testing & Deployment
- Initial tests include sending JSON formatted events from Databricks to Event Hubs and verifying event ingestion via the Event Hubs data explorer.
- For deployment, ensure that all necessary libraries (e.g., `azure-eventhub` for Databricks or dependencies listed in `requirements.txt` for Azure Functions) are correctly configured.

### Development Tools
- **Visual Studio Code:** Used with the Azure Functions extension for developing and deploying function apps.
- **Azure Portal:** Utilized for RBAC assignments, configuring Key Vault, and monitoring services.
- **Microsoft Fabric:** Used for real-time analytics, data streaming, and visualization.
- **draw.io:** Utilized for creating and maintaining architecture diagrams.
- **Postman:** Used for testing API endpoints and verifying data ingestion.

## Conclusion
This project leverages a blend of Azure and Microsoft Fabric services to build a robust, real-time weather monitoring and alert system. By evaluating both Azure Functions and Azure Databricks for data ingestion, the solution ensures an optimal balance between cost, performance, and scalability. The integration of secure secret management, real-time analytics, and automated alerts creates a comprehensive system suitable for continuously tracking weather conditions and responding to severe events.

## Social Media
- Connect with me on LinkedIn: [Ali Ghafarian](https://www.linkedin.com/in/ali-ghafarian-data-analyst)

## Gallery
### Cost Breakdown
![Azure Services Cost Breakdown](Images/cost.png)

### Email Alert Example
![Weather Alert Email Sample](Images/alert-email.png)
