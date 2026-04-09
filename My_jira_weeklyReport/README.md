<div align="center">

![UiPath](https://img.shields.io/badge/UiPath-Robotic_Enterprise_Framework-FA4616?style=for-the-badge&logo=uipath&logoColor=white)
![Jira](https://img.shields.io/badge/Jira-Integration-0052CC?style=for-the-badge&logo=jira&logoColor=white)

</div>

# 🤖 Jira Sprint Analytics & Automated Reporting Bot

This project is a high-performance Robotic Process Automation (RPA) solution developed using the **UiPath Robotic Enterprise (RE) Framework**. It automates the extraction of sprint data from **Jira**, performs deep risk analysis on "Carry-Over" items, and generates a dynamic HTML executive report sent via email to stakeholders.

## 📖 Project Overview

The automation identifies operational bottlenecks by analyzing sprint health. It specifically tracks:
* **Completion Rates:** Percentage of work finished vs. planned.
* **Carry-Over Risks:** Items moving to the next sprint, categorized by status (e.g., "To Do" vs "In Progress").
* **Critical Alerts:** Identification of highest-priority tasks that are unassigned or stalled (no updates for 3+ days).
* **Resource Distribution:** Analysis of workload across the team.

## 🚀 How It Works (Process Workflow)

1. **Initialization:** Loads configuration from `Config.xlsx`, initializes Jira Integration Service connections, and ensures a clean environment by killing legacy processes.
2. **Data Extraction:** Executes multiple **JQL (Jira Query Language)** searches to retrieve "Open", "Closed", and "Critical/Overdue" issues.
3. **Data Transformation:** Uses **LINQ** and **DataTable** manipulation to calculate sprint metrics, identify the most/least assigned users, and detect stalled workflows.
4. **Report Generation:** Dynamically builds a professional **HTML5/CSS3** report. The report header color changes based on sprint health (Green: Healthy, Yellow: Stalled, Red: Critical Risks).
5. **Distribution:** Attaches the generated HTML file and sends a formatted email using **GSuite/Gmail Integration**.

## 🛠️ Workflow Components

### 🏗️ Main.xaml
The core State Machine following REFramework best practices. It manages the high-level transitions between `Initialization`, `Get Transaction Data`, `Process Transaction`, and `End Process`.

### 🔍 Jira_ExtractData.xaml
Handles the communication with Jira via Integration Service.
* Performs JQL queries to fetch specific datasets.
* Outputs arrays of Jira Issue entities for processing.

### 📊 Data-Process.xaml
The analytical engine of the bot.
* **KPI Calculation:** Calculates completion rates using `Math.Round`.
* **Risk Logic:** Employs conditional logic to assign `Sprint_Health_Color`.
* **HTML Construction:** Merges analytical data into an embedded HTML template with inline CSS for cross-client email compatibility.

### 📧 Send_Mail.xaml
The communication module.
* Validates the existence of the report file before attempting to send.
* Uses `UIPATH.GSuite.Activities` to send high-priority emails with embedded HTML bodies and the report as an attachment.

## 📋 Input/Output Specifications

| Type | Name | Description |
| :--- | :--- | :--- |
| **Input** | `Config.xlsx` | Contains Jira Project Keys, Orchestrator Queue Names, and Email Recipient lists. |
| **Input** | `Jira JQL` | Dynamic queries based on sprint timelines (e.g., `updated >= -1w`). |
| **Output** | `Report_{Date}.html` | A standalone dashboard containing KPI cards and risk tables. |
| **Output** | `Email` | Sent to stakeholders with a summary of carry-over risks. |

## ⚙️ Configuration & Best Practices

* **Error Handling:** Wrapped in `Try-Catch` blocks within the REFramework to ensure that system exceptions trigger retries or graceful terminations.
* **Security:** All connections (Jira, Gmail) are managed through **UiPath Integration Service** connectors, removing the need for hardcoded credentials.
* **Scalability:** The use of `uiascb:issue_search_get_List[]` allows the bot to handle large volumes of Jira issues efficiently.

---