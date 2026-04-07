<div align="center">

![UiPath](https://img.shields.io/badge/UiPath-REFramework-FA4616?style=for-the-badge&logo=uipath&logoColor=white)
![Jira](https://img.shields.io/badge/Atlassian-Jira-0052CC?style=for-the-badge&logo=jira&logoColor=white)
![AI](https://img.shields.io/badge/OpenRouter-AI_Classification-black?style=for-the-badge&logo=openai&logoColor=white)

</div>

# 🤖 Jira Issue Intelligent Classifier & Labeler

This project is a high-performance **UiPath** automation designed to streamline project management by automatically classifying Jira issues using Large Language Models (LLMs). The workflow retrieves Jira tickets from an Orchestrator Queue, utilizes an AI model (via OpenRouter) to determine the intent, and updates the Jira labels accordingly.

## 📖 Project Overview

Managing a high volume of Jira tickets can be time-consuming for analysis experts. This automation acts as an **AI-powered Jira Analyst**. It reads the summary and description of incoming tickets and classifies them into one of three categories: `Bug`, `Feature`, or `Question`. Once classified, it automatically writes these labels back to the specific Jira issue using the Jira SDK.

## 🏗️ Technical Architecture

The project is built on the **Robotic Enterprise Framework (REFramework)**, ensuring high reliability, transaction-based logging, and professional error handling.

### Process Flow:
1. **Initialization:** Loads settings from `Data/Config.xlsx` and initializes API connections.
2. **Get Transaction:** Fetches a `QueueItem` containing Jira ticket details (Summary, Description, Key).
3. **Process (AI Classification):** Communicates with the LLM to categorize the issue.
4. **Action (Jira Update):** Applies the identified category as a label to the ticket in Jira.
5. **End Process:** Closes all connections and logs final status.

## 📂 Workflow Components

### 1. Main.xaml
The backbone of the automation. It manages the State Machine logic, handling transitions between Initialization, Transaction Retrieval, and Processing. It ensures that any **System Exception** triggers a retry or a clean shutdown.

### 2. AT.xaml (AI Tagger)
This module serves as the bridge between RPA and Artificial Intelligence.
* **Extraction:** Pulls `Summary` and `Description` from the `SpecificContent` of the Transaction Item.
* **LLM Integration:** Sends a POST request to **OpenRouter** (specifically using the `arcee-ai/trinity-large-preview` model).
* **Prompt Engineering:** Instructs the AI to act as a "Jira Analysis Expert" and return a strict JSON output.
* **JSON Parsing:** Deserializes the response to isolate the classification result.

### 3. eafae.xaml (Jira Labeler)
Handles the final interaction with the Jira platform.
* **Data Sanitization:** Employs **Regex** `(?<="category":\s")([^"]+)` to extract the clean label value from the AI's JSON string.
* **Jira Scope:** Establishes a secure connection using **API Token** authentication.
* **Method Execution:** Invokes the `SetLabelsAsync` method to update the Jira ticket dynamically.
* **Error Handling:** Features a robust **Try-Catch** block to catch connection failures or permission issues, throwing descriptive `SystemException` messages if the update fails.

## 📊 Input/Output Specifications

### Queue Item (SpecificContent)
| Key | Type | Description |
| :--- | :--- | :--- |
| `Summary` | String | The title/summary of the Jira ticket. |
| `Description` | String | Detailed content of the Jira ticket. |
| `Key` | String | Unique Jira identifier (e.g., "PROJ-123"). |

### Configuration (Config.xlsx / Assets)
| Name | Type | Description |
| :--- | :--- | :--- |
| `openroutherapi` | String/Asset | API Key for OpenRouter authorization. |
| `jiraToken` | Asset | Secure API Token for Jira access. |
| `jirauser` | String | Username/Email for Jira account. |
| `jiraserver` | String | The base URL of the Jira instance. |

## ⚙️ Configuration & Best Practices

* **Security:** All sensitive credentials (Jira Token, AI API Key) are retrieved via the `in_Config` dictionary, encouraging the use of **Orchestrator Assets** for credential security.
* **Resiliency:** The `NetHttpRequest` in the AI module is configured with a **Retry Policy** (3 attempts) to handle network timeouts or "Too Many Requests" (429) errors.
* **Regex Validation:** Using Regex for data extraction ensures that the label update doesn't fail due to unexpected white spaces or additional text provided by the LLM.
* **Logging:** Strategic `LogMessage` activities are placed at each milestone (e.g., "AI tagging started", "Jira scope entered") to provide full visibility in Orchestrator logs.
