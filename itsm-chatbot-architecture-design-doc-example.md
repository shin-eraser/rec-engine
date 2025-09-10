<p><a target="_blank" href="https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW" id="edit-in-eraser-github-link"><img alt="Edit in Eraser" src="https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fgithub%2FOpen%20in%20Eraser.svg?alt=media&amp;token=968381c8-a7e7-472a-8ed6-4a6626da5501"></a></p>

## Project Overview
The ITSM chatbot is an internal web application designed for JM Family employees to easily interact with IT services. Employees will use the chatbot to:

- Ask IT-related questions (e.g., “How can I provision a new laptop?”).
- Retrieve knowledge base articles from SharePoint.
- Open and manage IT support tickets through ServiceNow.
**Target users:** All JM Family employees across business units.

**Business objectives:**

- Improve self-service IT support.
- Reduce IT helpdesk workload.
- Provide quick, AI-powered responses.
- Streamline ticket creation and tracking.
---

## System Architecture
### High-Level Overview
[![JM Family ITSM Chatbot - High-Level System Architecture](https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW/preview?elements=3fX88YNucuzUIjwa6XzWjQ&type=embed)](https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW?elements=3fX88YNucuzUIjwa6XzWjQ)

### Major Components
- **Web Application**
 Intranet-only, browser-based interface for employees. Provides chat interface, authentication integration, and secure communication with backend services.
- **Azure OpenAI**
 Processes natural language queries, generates responses, and determines intent (knowledge retrieval vs. ticket creation).
- **SharePoint Integration**
 Acts as the knowledge repository. The chatbot retrieves and surfaces relevant articles/documents.
- **ServiceNow Integration**
 Handles ITSM workflows for ticket creation, updates, and tracking.
- **Authentication & Access Control**
    - Integrated with Azure AD (SSO).
    - Role-based access to limit ticket management features to employees vs. IT staff.
- **Network & Hosting**
    - Hosted in JM Family’s intranet network (private endpoints, no public exposure).
    - Web App + APIs hosted in Azure App Service or Azure Kubernetes Service (AKS).
    - All connections secured via HTTPS and Azure Private Link.
---

## Data Flow
[![ITSM Chatbot Data Flow](https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW/preview?elements=TaH9s-yASJDKvCF_vinLcw&type=embed)](https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW?elements=TaH9s-yASJDKvCF_vinLcw)

1. User submits a query in the chatbot UI.
2. Web App sends query to Azure OpenAI for intent recognition.
3. If query is informational: OpenAI retrieves relevant documents from SharePoint.
4. If query is service-related: OpenAI triggers ServiceNow API call to create/update tickets.
5. Response returned to user via Web App interface.
---

## Integration Points
- **SharePoint**
 REST API and Microsoft Graph API for retrieving knowledge articles and documents.
- **ServiceNow**
 REST API integration for creating, updating, and retrieving ticket status.
- **Azure OpenAI**
    - GPT-based models for intent classification and response generation.
    - Prompt engineering with contextual grounding from SharePoint content.
---

## CI/CD Pipeline
[![ITSM Chatbot CI/CD Pipeline](https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW/preview?elements=lWsDnvLHVn1TrsGPmWKVyQ&type=embed)](https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW?elements=lWsDnvLHVn1TrsGPmWKVyQ)

- **Source Control:** GitHub repositories for code, prompts, and deployment configurations.
- **Build & Test:**
    - Automated builds triggered on pull requests.
    - Unit, integration, and security testing in pipeline.
- **Deployment:**
    - CI/CD pipelines using GitHub Actions or Azure DevOps.
    - Automated deployment to Azure environments.
- **Environment Promotion:**
    - Dev → Test → Prod with approvals at each stage.
    - Canary/blue-green deployment options for production.
- **Rollback:**
    - Automated rollback to previous version if deployment fails or health checks fail.
---

## Security and Compliance
- **Data Privacy & Access Control**
    - Queries anonymized and logged without PII where possible.
    - Azure AD SSO enforces user identity verification.
- **Sensitive Information Handling**
    - ServiceNow tickets may include confidential IT details → secured with TLS and role-based access.
- **Compliance**
    - Adheres to JM Family IT security policies.
    - Aligns with SOC 2 and ISO 27001 security principles.
---

## Monitoring and Logging
- **Health Monitoring:** Azure Monitor + Application Insights for uptime and performance.
- **Error Logging:** Centralized log collection in Azure Log Analytics.
- **Usage Analytics:** Track query types, resolution success, and ticket volume.
- **Alerting:** Automated alerts for failures, performance degradation, and security anomalies.
---

## Feedback and Improvement Strategy
[![ITSM Chatbot Feedback & Improvement Cycle](https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW/preview?elements=06J4Se6TZEZZrZRRy-8Rzg&type=embed)](https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW?elements=06J4Se6TZEZZrZRRy-8Rzg)

- **Feedback Collection:**
    - Inline thumbs-up/down in chatbot responses.
    - Survey links in ServiceNow tickets.
- **Review & Implementation:**
    - IT team reviews logs and feedback weekly.
    - Continuous improvement cycle for prompts and integrations.
- **Knowledge Base Updates:**
    - Automatic flagging of “no answer” queries for SharePoint content team.
    - Quarterly content review with IT stakeholders.
- **Model Tuning:**
    - Continuous prompt optimization.
    - Evaluation of Azure OpenAI model updates for performance improvements.



<!-- eraser-additional-content -->
## Diagrams
<!-- eraser-additional-files -->
<a href="/itsm-chatbot-architecture-design-doc-example-JM Family ITSM Chatbot - High-Level System Architecture-1.eraserdiagram" data-element-id="voA3x9-0gJ0ZV51bzZX_K"><img src="/.eraser/PnvLGz7cS5SMCgqLCPyW___67Ed9tq1O9R1Ue4HoW2lN54ivGo2___---diagram----36a2d5caaea5114ae6424a60a65a4be2-JM-Family-ITSM-Chatbot---High-Level-System-Architecture.png" alt="" data-element-id="voA3x9-0gJ0ZV51bzZX_K" /></a>
<a href="/itsm-chatbot-architecture-design-doc-example-ITSM Chatbot Data Flow-2.eraserdiagram" data-element-id="eew6gvS6KyNXFriqE7_rq"><img src="/.eraser/PnvLGz7cS5SMCgqLCPyW___67Ed9tq1O9R1Ue4HoW2lN54ivGo2___---diagram----09cc5d324c46ede62ea9bb1fe189e926-ITSM-Chatbot-Data-Flow.png" alt="" data-element-id="eew6gvS6KyNXFriqE7_rq" /></a>
<a href="/itsm-chatbot-architecture-design-doc-example-ITSM Chatbot CI/CD Pipeline-3.eraserdiagram" data-element-id="FnJGQhdo6K0gSgiWWD5Xn"><img src="/.eraser/PnvLGz7cS5SMCgqLCPyW___67Ed9tq1O9R1Ue4HoW2lN54ivGo2___---diagram----ea39aa62791d6f6c3a74f50dd1296021-ITSM-Chatbot-CI-CD-Pipeline.png" alt="" data-element-id="FnJGQhdo6K0gSgiWWD5Xn" /></a>
<a href="/itsm-chatbot-architecture-design-doc-example-ITSM Chatbot Feedback & Improvement Cycle-4.eraserdiagram" data-element-id="_WE1sqAcRoVPAbsklx6sM"><img src="/.eraser/PnvLGz7cS5SMCgqLCPyW___67Ed9tq1O9R1Ue4HoW2lN54ivGo2___---diagram----793804e315e921d27726f53db5fbbf67-ITSM-Chatbot-Feedback---Improvement-Cycle.png" alt="" data-element-id="_WE1sqAcRoVPAbsklx6sM" /></a>
<!-- end-eraser-additional-files -->
<!-- end-eraser-additional-content -->
<!--- Eraser file: https://jmfamily.eraser.io/workspace/PnvLGz7cS5SMCgqLCPyW --->