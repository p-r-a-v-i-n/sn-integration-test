# ServiceNow - GitHub Bi-Directional Integration Hub

## 📌 Project Overview
This project establishes a seamless, bi-directional integration between **ServiceNow** and **GitHub**. It automates the lifecycle of a technical issue by synchronizing GitHub Issues with ServiceNow Incidents.

### Key Logic Flow
1. **GitHub to ServiceNow:** A Webhook triggers a Scripted REST API in ServiceNow to create a new Incident when an issue is opened in GitHub.
2. **ServiceNow to GitHub:** When the Incident is resolved, ServiceNow automatically posts a comment to the GitHub issue and closes it.

---

## ⚙️ Technical Architecture



### 1. Inbound Integration (Scripted REST API)
- **Path:** `/api/x_1642813_github_0/github_integration`
- **Logic:** Extracts the GitHub Issue number, Title, and Description from the JSON payload.
- **Data Mapping:**
  - `short_description` = GitHub Title
  - `description` = GitHub Body
  - `u_github_number` = GitHub Issue Number

### 2. Outbound Integration (REST Message V2)
- **Method:** `POST` (Add Comment) and `PATCH` (Close Issue).
- **Endpoint:** `https://api.github.com/repos/p-r-a-v-i-n/sn-integration-test/issues/${issue_num}`
- **Conditional Logic:** The script verifies a `201` status code from the comment action before attempting to close the issue, ensuring no data loss.

---

## 🛠️ File Structure
The following components are included in the uploaded **Update Set**:
- **Scripted REST API:** `GitHub Inbound Handler`
- **REST Message:** `Github API` (Methods: Add Comment, Close Issue)
- **Flow Designer:** `Sync GitHub Resolution`
- **Action Designer:** `GitHub Close & Comment Action` (Custom Script Step)

---

## 📜 Integration Script Logic
The core logic utilizes the `sn_ws.RESTMessageV2` API to handle sequential updates:

```javascript
// Step 1: Add Comment
var rComment = new sn_ws.RESTMessageV2('Github API', 'Add Comment');
var responseComment = rComment.execute();

// Step 2: If Success, Close Ticket
if (responseComment.getStatusCode() == 201) {
    var rClose = new sn_ws.RESTMessageV2('Github API', 'Close Issue');
    rClose.execute();
}
```
---

## Future Roadmap: Security Hardening
The current version uses token-based authentication in the REST headers. The planned next phase includes:
- **Connection & Credential Aliases**: Moving secrets from code to the ServiceNow Credential Store (`sys_auth_profile`).
- OAuth 2.0: Implementing GitHub App-based authentication for enhanced security.

---

## How to Install
- Import the provided Update Set XML into your ServiceNow instance.
- Preview and Commit the Update Set.
- Update the `Authorization` header in the REST Message with your GitHub Fine-Grained Personal Access Token (PAT).
- Configure your GitHub Repository Webhook to point to the ServiceNow Scripted REST API endpoint.

---

Developed as a ServiceNow-DevOps Integration Proof of Concept.