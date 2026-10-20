# Workshop Prerequisites

> **Read this before the session.** The workshop is **hands‑on** — you'll build a real Genie Agent in your own sandbox. The setup below must be done **ahead of time** so we don't spend session time troubleshooting access. If any step fails, contact your Databricks workspace administrator / platform team **before** the workshop.

## What you need

| # | Requirement | How to check |
|---|-------------|--------------|
| 1 | **A Databricks account** | You can sign in to Databricks (most staff have an account via central identity/SSO). |
| 2 | **Access to a sandbox workspace** | You can open a workspace and see the notebook UI. *An account alone is not enough — you also need workspace access.* |
| 3 | **Workspace entitlement** | You can create and run notebooks. |
| 4 | **Databricks SQL entitlement** | You can access SQL / run queries (needed to create tables and Genie Agents). |
| 5 | **A running SQL warehouse** | A **Serverless** or **Pro** SQL warehouse you can use (`CAN USE`). Genie requires Pro/Serverless. |
| 6 | **Unity Catalog privileges** | In **your own schema** in the shared **`containerunitycatalog`** catalog, you can `CREATE TABLE`, `CREATE VOLUME`, and `CREATE FUNCTION`. By default the workshop uses a schema named after your login (e.g. `ashish.arun@…` → `ashish_arun`). |
| 7 | **Genie enabled** | The workspace has **Genie / Genie Agents** enabled (ask your admin if unsure). |
| 8 | **Serverless notebook compute** (or a UC‑enabled cluster) | You can attach a notebook to compute. |

> **Consumers (non‑builders) welcome.** If you're joining to learn how to *use* and *give feedback on* Genie Agents rather than build them, you still need items 1–3 so you can follow along and try the agent.

## Before the session
1. **Confirm access** (items 1–8 above). If anything is missing, request it from your platform/entitlements team now — access requests can take time.
2. **Import the workshop assets** into your workspace so the folders sit side‑by‑side:
   ```
   /Workspace/Users/<your_email>/genie-agents-workshop/
     notebooks/     ← workshop notebooks (00–06, 08)
     templates/     ← Genie Agent configuration
   ```
3. **Open `00_workshop_config`** and check the **catalog** (`containerunitycatalog`) and **schema** (defaults to your login name). If your schema has a different name, change the **Schema** widget.

## During & after the session
- You'll create Genie Agents and supporting tables in **your own sandbox**.
- **Clean up when you're done.** This version doesn't include a cleanup notebook. Delete your two Genie agents (Baseline and Knowledge Store) from the Genie UI. Drop the workshop tables in your schema only if your platform team asks you to.

## Quick self‑test (optional)
In a notebook, run:
```python
from databricks.sdk import WorkspaceClient
w = WorkspaceClient()
print("Host:", w.config.host)
print("User:", w.current_user.me().user_name)
print("Warehouses:", [wh.name for wh in w.warehouses.list()][:5])
```
If this prints your host, user, and at least one warehouse, you're ready.
