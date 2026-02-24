
# Marketo Self-Service Flow Step (SSFS) for OpenAI via n8n

This repository contains an n8n workflow template designed to create a  **Marketo Self-Service Flow Step (SSFS)** . This allows you to send data from Marketo Smart Campaigns directly to OpenAI (GPT models) and write the responses back to Marketo lead fields or activity logs automatically.

## 🚀 Overview

The workflow acts as a bridge between Marketo and OpenAI, following the SSFS protocol. It handles:

* **Service Installation** : Providing the openAPI swagger definition (/install) and service definition (/getServiceDefinition) to Marketo.
* **Icon Serving** : Serving the brand and service icons. This can use any publicly available icon set in the SSFS Config step.
* **Asynchronous Processing** : Receiving a batch of leads, acknowledging receipt immediately to Marketo, processing prompts via OpenAI, and then using a callback to return the results.

## 🛠 Prerequisites

* An **n8n** instance (self-hosted or cloud) with a public-facing URL.
* A **Marketo** instance with Admin access to "Service Providers."
* An  **OpenAI API Key** .

## 📦 Setup Instructions

### 1. Import the Workflow to n8n

There are two ways to import this template into your n8n instance:

* **Option A: Import from File**
  1. Download the `n8n-workflow.json` file from this repository.
  2. In n8n, go to the **Workflows** tab and select  **Import from File** .
* **Option B: Import from URL**
  1. In n8n, select  **Import from URL** .
  2. Use the following jsDelivr CDN link:
     `https://cdn.jsdelivr.net/gh/jdarring/marketo-ssfs-n8n-openai/n8n-workflow.json`

### 2. Configure OpenAI Credentials

1. Find the node named `OpenAI2` (or search for the OpenAI node).
2. Click on the **Credentials** dropdown.
3. Select **Create New** (if you haven't already) and enter your OpenAI API Key.
4. Ensure the model (e.g., `gpt-4o-mini` or `gpt-4`) matches your requirements.

### 3. Customize the SSFS Configuration

The node named `SSFS Config` (a "Set" node near the start of the workflow) contains the metadata that Marketo uses to display your service. You **must** adjust these values:

* **`apiName`** : A unique internal name for your service (e.g., `openai-lead-scoring`).
* **`provider`** : Your company name.
* **`apiKey`** : Inside the JSON, find the `apiKey.value`. This is the shared secret you will enter in Marketo during installation to secure your webhook.
* **`flow`** : Update the `name`, `filterName`, and `triggerName` to change how the step appears in the Marketo UI.

### 4. Activate the Webhook

1. Click the **Webhook** node at the start of the workflow.
2. Note the  **Production URL** . It usually looks like `https://your-n8n-instance.com/webhook/[UNIQUE-ID]/:path`.
3. Click **Execute Workflow** once to prepare for the first incoming request, then set it to  **Active** .

## 🔌 Endpoints

The workflow uses the `:path` parameter in the n8n Webhook node to route requests based on the SSFS specification. Below are the endpoints exposed by the workflow:

| Endpoint                  | Method   | Description                                                                                       |
| ------------------------- | -------- | ------------------------------------------------------------------------------------------------- |
| `/install`              | `GET`  | Returns the OpenAPI Swagger definition required for initial service installation in Marketo.      |
| `/getServiceDefinition` | `GET`  | Returns the detailed metadata describing flow attributes, global attributes, and callback fields. |
| `/status`               | `GET`  | Marketo polls this nightly to check the health and status of the service.                         |
| `/serviceIcon`          | `GET`  | Serves the 32x32 pixel icon displayed in the Smart Campaign flow palette.                         |
| `/brandIcon`            | `GET`  | Serves the larger brand icon displayed in the Service Providers menu.                             |
| `/submitAsyncAction`    | `POST` | The primary endpoint where Marketo sends lead data batches for OpenAI processing.                 |
| `/getPicklist`          | `POST` | (Optional) Returns choices for dynamic dropdown menus in the Flow Step UI.                        |

## 📥 Installation in Marketo

1. Log in to Marketo and go to **Admin** >  **Service Providers** .
2. Click  **Create New Service** .
3. **Service Settings** :

* **Service Name** : OpenAI Text Generator (or your choice).
* **API URL** : Your n8n Production Webhook URL  **including the /install path** .
  * *Example:* `https://n8n.example.com/webhook/[UNIQUE-ID]/install`

1. **Authentication** :

* Select  **API Key** .
* **Header Name** : `x-api-key`.
* **API Key** : The value you set in the `SSFS Config` node.

1. **Incoming Fields** :

* Map the `response` attribute to the Marketo Lead Field where you want the AI output stored.

1. Click  **Save** . Marketo will use the `/install` URL to fetch the service definition and verify the setup.

## 🔄 How It Works (The Logic)

1. **Marketo calls `submitAsyncAction`** : Marketo sends a list of leads and their prompt data.
2. **n8n Responds 202** : The workflow immediately returns an "Accepted" status to Marketo so the Smart Campaign isn't held up.
3. **Async Processing** :

* The workflow splits the lead batch.
* For each lead, it calls the **OpenAI** node.
* It aggregates the results back into a JSON array.

1. **Callback** : The workflow sends the final results back to Marketo's `callbackUrl` provided in the original request.

## ⚠️ Security Note

This workflow uses a simple `x-api-key` header check in the `Check API Key` node. Ensure your n8n instance is running over HTTPS to keep this key and your OpenAI key secure.

## 📄 License

This template is provided "as-is" under the MIT License.
