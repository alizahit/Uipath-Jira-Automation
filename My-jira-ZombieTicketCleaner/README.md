<div align="center">

![UiPath](https://img.shields.io/badge/UiPath-Studio-FA4616?style=for-the-badge&logo=uipath&logoColor=white)
![Jira](https://img.shields.io/badge/Jira-Integration-0052CC?style=for-the-badge&logo=jira&logoColor=white)
![Excel](https://img.shields.io/badge/Excel-107C41?style=for-the-badge&logo=microsoftexcel&logoColor=white)

</div>

# 🤖 Jira Zombie Ticket Cleanup & Organizer

## 1. Project Overview
This project is an advanced Robotic Process Automation (RPA) solution developed in **UiPath** to maintain data hygiene in Jira Scrum/Kanban projects. It is designed to identify "Zombie" (abandoned/forgotten) tasks, organize unowned issues, and protect the overall health of the Jira database.

Unlike native Jira automation, this bot uses Jira purely as a data source. It makes intellectual decisions by cross-referencing external data (an Excel-based Employee List) to dynamically reassign tasks or safely close them, preventing hundreds of forgotten tasks from polluting project reports and backlogs.

## 2. Technical Architecture
The project is built on an **Init-Process (Refrigerator Model)** architecture utilizing UiPath Orchestrator. 

**A. Init Phase (Data Acquisition):**
The bot starts by scanning Jira using a strict JQL query to prevent falsely flagging future tasks as "Zombies".
* **Condition Criteria:**
    * **Project:** Target Project (e.g., SCRUM)
    * **Status:** All statuses except "Done"
    * **Age:** Last updated > 90 days ago
    * **Start Date:** Before `now()` (ignores tasks planned for future sprints)
* The resulting Issue List is pushed to an Orchestrator Queue (`JiraCleanupQueue`) using the `IssueKey`. This queue acts as a transaction log and backup mechanism.

**B. Process Phase (Business Logic):**
Each transaction is fetched from the queue. The bot pulls live details of the issue and executes the following logic:
1.  **Employee Validation:** Checks the Assignee's email against the external active employee list (Excel). If the employee has left the company, the task is dynamically **reassigned** to an active employee.
2.  **Cleanup & Archiving:** If the employee is active but the task is classified as "Low" priority and severely outdated, the bot changes the status to **"Done"** and leaves an automated comment: *"Closed due to cleanup, please reopen if necessary"*, leaving room for manual intervention.

## 3. Workflow Components

The automation runs sequentially through the following modules:

### 🧹 `clearQueue.xaml`
Ensures a clean slate for the current run. Uses a `Retry Scope` to fetch and delete any leftover items from `JiraCleanupQueue` until the count reaches 0.

### 🔍 `serachTicket.xaml`
Establishes the Jira connection and extracts the raw issue data.
* **Code Snippet (JQL):**
    ```sql
    project = 'SCRUM' AND status != Done AND updated <= -90d AND StartDate < now()
    ```
* **Output:** Returns an array of Jira issues (`out_AllOpenIssues`).

### 📊 `readExcel.xaml`
Reads the local HR dataset.
* **Action:** Loads the active personnel list into a DataTable (`dt_Employees`). This allows flexible Active Directory-like validation without complex infrastructure.

### 📥 `addQueue.xaml`
Acts as the logging and safety mechanism.
* **Action:** Iterates through the fetched Jira issues and pushes each `IssueKey` into the Orchestrator Queue. Uses `DateTime.Now` for transaction referencing.

### ⚙️ `filterTicket.xaml`
The core intellectual engine of the bot.
* **Logic:** Executes the cross-referencing logic against `dt_Employees`. 
* **Actions Performed:** * Validates assignees.
    * Reassigns orphaned tickets (Load Balancing).
    * Closes low-priority zombie tickets via API and adds the manual-intervention comment.

## 4. Input/Output Specifications

| Component | Data Input | Data Output |
| :--- | :--- | :--- |
| **`serachTicket`** | JQL String | `List<JiraIssue>` |
| **`readExcel`** | Local File Path (`.xlsx`) | `dt_Employees` (DataTable) |
| **`addQueue`** | `List<JiraIssue>` | Orchestrator Queue Items (Logs) |
| **`filterTicket`** | `dt_Employees`, `JiraIssue` | API POST/PUT (Reassign/Close) |

## 5. Configuration & Best Practices

**Configuration:**
The project relies on a `Config` dictionary/file for seamless environment migration:
* `OrchestratorQueueName`: Set to `"JiraCleanupQueue"`.
* `EmployeeExcelPath`: Path to the HR active employee spreadsheet.
* `Jira_ProjectKey`: The target project for the cleanup operation.

**Best Practices & Security:**
* **No Hardcoded Credentials:** Jira API authentication is handled securely via UiPath Integration Service/Orchestrator Assets.
* **Error Handling:** API calls (like adding comments or changing statuses) are wrapped in `Try-Catch` blocks to prevent the bot from crashing if Jira times out.
* **Audit Trail:** Utilizing Orchestrator Queues merely as a transaction log ensures there is a permanent corporate memory of which tickets were evaluated and processed.

**Limitations & Risks:**
* **Date Dependency:** The bot relies heavily on the accurate management of the `StartDate` and `Updated` fields in Jira. Malformed dates might lead to unintended closures.
* **Priority Judgment:** Closing "Low" priority tasks is automated without manager approval. However, the automated Jira comment mitigates this risk by explicitly allowing manual reopening.