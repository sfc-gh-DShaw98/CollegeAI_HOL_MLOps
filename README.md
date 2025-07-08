# College of AI MLOps HOL: Operationalize SageMaker Models with Snowflake Model Registry for Scalable Batch Inferences
**Connect, register, and run inference at scale—no UDFs or endpoints required.**

![Lab Architecture](images/MLOpsHOLWorkflow.jpg)

---

## 🎬 Lab Overview Video
Watch the [8-minute Lab Overview Video](https://drive.google.com/file/d/1_wWRvwZu1_Oi_2vRRVWkaX3-hp1C8qi0/view?usp=sharing) for a quick walkthrough of the 3-phase MLOps HOL workflow.

Below, we outline the three major phases in this HOL. Each phase builds on the previous, moving from SageMaker training to full ML lifecycle operations inside Snowflake.

🔽 Jump to:
- [🛠️ Hands-On Lab Overview](#hands-on-lab-overview)
- [📂 Repository Structure](#repository-structure)
- [⚠️ Troubleshooting & FAQ](#troubleshooting--frequently-asked-questions-faq)
- [🧹 Cleanup](#cleanup--cost-stewardship-procedures)

---
<a name="hands-on-lab-overview"></a>
## 🛠️ Hands-On Lab Overview

In this hands-on lab, you’ll step into the shoes of an **MLOps engineer** tasked with **operationalizing a predictive model trained in Amazon SageMaker—but all the customer data lives in Snowflake**. Instead of rewriting model logic or deploying a real-time endpoint, you’ll register the model in Snowflake’s Model Registry and run batch inference at scale using warehouse compute.

### 📋 What You’ll Do:
- **Phase 1: [Model Development & SageMaker Integration](lab_instructions/phase1_setup.md)**
  - **Train** and evaluate an XGBoost model in **SageMaker**
  - **Log** the model into [**Snowflake Model Registry**](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/overview) 
  - **Perform batch-scoring** on historical mortgage data stored **in Snowflake** (Weeks 1–5)

- **Phase 2: [Model Monitoring with Snowflake ML Observability](lab_instructions/phase2_observability.md)**
  - **Configure** [**Snowflake ML Observability**](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/model-observability) to baseline, monitor performance, and detect drift
  - **Configure** [**Snowflake Alerts**](https://docs.snowflake.com/en/guides-overview-alerts) to send email notification for performance degradation
  - **Generate custom** performance reports and visualizations

- **Phase 3: [End-to-End Model Retraining in Snowflake ML](lab_instructions/phase3_retraining.md)**
  - **Preprocess** updated training data using [**Snowpark ML**](https://docs.snowflake.com/en/developer-guide/snowpark-ml/index) (e.g. OneHotEncoder)
  - **Register features and entities** in the [**Snowflake Feature Store**](https://docs.snowflake.com/en/developer-guide/snowflake-ml/feature-store/overview)
  - **Train** a new binary classification **model natively in Snowflake**
  - **Log** the new model version to the [**Snowflake Model Registry**](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/overview)
  - **Use** [**Snowflake ML Explainability**](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/model-explainability) to interpret the model
  - **Monitor drift** with [**Snowflake ML Observability**](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/model-observability) between V1 and V2
  - **Promote** V2 to production using [**model version aliases**](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/overview#model-version-aliases)
  - **Run version-agnostic batch scoring** or [inference in Snowflake warehouses](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/warehouse) using the [**PRODUCTION alias**](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/overview#model-version-aliases)

---

## 📖 Table of Contents

- [Why this Matters](#why-this-matters)
- [Customer Discovery Questions](#customer-discovery-questions)
- [Repository Structure](#repository-structure)
- [Prerequisites & Setup Details](#prerequisites--setup-details)
- [Estimated Lab Timeline](#estimated-lab-timeline)
- [Lab Completion Requirements](#lab-completion-requirements)
- [Placeholder & Naming Conventions](#placeholder--naming-conventions)
- [Troubleshooting & FAQ](#troubleshooting--frequently-asked-questions-faq)
- [Cleanup & Cost-Stewardship Procedures](#cleanup--cost-stewardship-procedures)
- [Links to Internal Resources & Helpful Documents](#links-to-internal-resources--helpful-documents)
- [Author & Support](#author--support)

---
<a name="why-this-matters"></a>
## 📌 Why this Matters

- **Centralized governance:** Register and version SageMaker models in Snowflake
- **Frictionless inference:** Score Snowflake data—no UDFs, containers, or endpoints required
- **Scalable batch scoring:** Leverage warehouse compute for performance and elasticity
- **ML-Analytics collaboration:** Empower analysts to explore predictions and metrics using SQL
- **Production-grade monitoring:** Detect drift, retrain, and promote models—all in Snowflake

---
<a name="customer-discovery-questions"></a>
## 🧭 Customer Discovery Questions

Provide **5 to 6 open-ended questions** for customer conversations related to this HOL.

- "Do you train models in SageMaker or another platform while your inference data lives in Snowflake?"
- "How are you managing model lifecycle and versioning across platforms today?"
- "Is batch scoring a requirement for your business processes (e.g., sub-hourly/hourly/nightly/weekly runs)?"
- "How do you evaluate or monitor model performance in production?"
- "Would centralizing models and data improve collaboration between data teams?"
- "How long does it take to get a typical new model into production?"
- "How do you manage model promotion that is creating a new model version to replace an older one in a production pipeline?"

---
<a name="repository-structure"></a>
## 📂 Repository Structure

```bash
├── README.md           # Main entry point
├── config/             # Configuration templates, credentials
├── code/               # SQL/Python scripts for automation
├── notebooks/          # Interactive Jupyter notebooks
├── data/               # Datasets (CSV, JSON) or external links
├── images/             # Diagrams and visual assets
├── lab_instructions/   # Step-by-step detailed instructions
│ ├── phase1_setup.md
│ ├── phase2_ml_observability.md
│ └── phase3_retraining.md
└── troubleshooting/    # Common issues and resolutions
│   └── faq.md
```
---
<a name="prerequisites--setup-details"></a>
## ✅ Prerequisites & Setup Details

**Snowflake Environment:**
- Account with ACCOUNTADMIN privileges to create:
  - Databases, schemas, warehouses, and stages
  - Service user (mlops_user) with key-pair authentication
  - Email notification integration
  - [See detailed setup instructions](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/lab_instructions/phase1_setup.md)
    
**AWS SageMaker:**
- Access to SE-Sandbox or equivalent environment
- JupyterLab space with SageMaker Distribution 2.4.2
- **Important:** Always STOP your JupyterLab space when not actively working and DELETE it after completing the lab
- [See detailed setup instructions](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/lab_instructions/phase1_setup.md)
  
**Required Files**
- **Data Files:**
  - [Mortgage_Data.csv](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/data/Mortgage_Data.csv) (for initial model training)
  - [NewTrainingData.csv](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/data/NewTrainingData.csv) (for model retraining)
    
**Configuration Files:**
  - [connections.toml](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/config/connections.toml) (Snowflake connection config)
  - rsa_private_key.pem (for secure authentication)
  - [environment.yaml](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/config/environment.yml) (for Phase 3 SPCS packages)
  - [College-of-AI-MLOPsExerciseNotebook.ipynb](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/notebooks/College-of-AI-MLOPsExerciseNotebook.ipynb) (SageMaker notebook)
  - [MLOPs Snowflake ML Observability In Action Exercise.ipynb](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/notebooks/MLOPs%20Snowflake%20ML%20Observability%20In%20Action%20Exercise.ipynb) (Snowflake notebook)
  - [MLOPs End-to-End Snowflake ML Retraining Exercise.ipynb](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/notebooks/MLOPs%20End-to-End%20Snowflake%20ML%20Retraining%20Exercise.ipynb) (Snowflake notebook)

**Notebook Environments**
- **Phase 1:** AWS SageMaker JupyterLab with Python 3.8+
- **Phase 2:** Snowflake Notebooks with Warehouse Runtime
- **Phase 3:** Snowflake Notebooks with Container Runtime (Snowflake ML Runtime CPU 1.0)

---
<a name="estimated-lab-timeline"></a>
## ⏲️ Estimated Lab Timeline

Provide a brief agenda to help SEs understand pacing:

- **Phase 1 (Env setup, model training, model registry, & batch inference):** ~45 min
- **Phase 2 (Model obserability, monitor drift, & generate alerts):** ~30 min
- **Phase 3 (End-to-end model retraining, model explainability, model promotion & cleanup):** ~30 min

<a name="lab-completion-requirements"></a>
## Lab Completion Requirements

To receive credit for completing this HOL, you must successfully complete all DORA evaluations:
- SEAI50: Confirms SageMaker model registration in Snowflake
- SEAI51: Confirms batch inference completion
- SEAI52: Verifies ML Observability setup
- SEAI53: Confirms Model Version V2 registration
- SEAI54: Verifies PRODUCTION alias assignment to V2

These are automatically tracked through **DORA grading functions embedded** in each notebook. You must run the grading cells and see confirmation for each SEAI step.

Check the status of your DORA evaluations using this [Tableau report](https://us-east-1.online.tableau.com/vizportal/api/saml/login?dest=%2F%23%2Fsite%2Fsnowflake%2Fviews%2FSEUniversity%2FCollegeCertifications%3F%253Aiid%3D1&target_site=snowflake&forceSitePick=false&sendPodInfo=false&authSetting=&siteLuid=&embedded=false&idpConfigurationId=)

---
<a name="placeholder--naming-conventions"></a>
## 🔖 Placeholder & Naming Conventions

- Databases/schemas: `college-of-ai`/`public`
- SageMaker instance name: `college-of-ai-<your name>` 
- Model versions: `COLLEGE_AI_HOL_<MODEL_NAME>_vX`
- Secrets management: Safely store credentials and API keys (never commit to GitHub).

---
<a name="troubleshooting--frequently-asked-questions-faq"></a>
## ⚠️ Troubleshooting & Frequently Asked Questions (FAQ)

These are the most common blockers and questions SEs encounter when completing the College of AI: MLOps with SageMaker & Snowflake Model Registry HOL.

**1. 🔐 Authentication & Snowpark Connection Errors**
  - **Error:** `Failed to connect` to DB or `Authentication attempt rejected by the current authentication policy`
  - **Cause:** Invalid JWT token, IP not allowed, or misconfigured `connections.toml`
  - **Tips:**
    - Use **`mlops_user`** inside SageMaker (not your SE Snowflake user).
    - Ensure your IP is correctly added to your network policy allowlist: run `!curl ifconfig.me` and check Snowflake's network policy.
    - Confirm your `connections.toml` file has the correct path, and the key includes the full `BEGIN PRIVATE KEY / END PRIVATE KEY` block.
    - If using key-pair auth, **delete your existing key files** and regenerate both keys using:
      ```bash
      openssl genrsa -out rsa_private_key.pem 2048
      openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
      ```
    - Be sure your public key includes the full:
      ```bash
      ALTER USER mlops_user SET RSA_PUBLIC_KEY='
      -----BEGIN PRIVATE KEY-----
      ...
      -----END PRIVATE KEY-----';
      ```
    - **Download the private key file** and upload it directly into your SageMaker instance to avoid formatting issues. Don’t try to copy/paste from a preview or browser view.

**2. ☁️ SageMaker Access & Sandbox Issues**
  - **Error:** `Permission denied` when loading data from S3
  - **Cause:** Running in the wrong AWS account (e.g., onboarding or capstone sandbox)
  - **Fix:** Use the **AWS SE Sandbox** (not onboarding or se-capstone). If needed, submit a [**LIFT**](https://lift.snowflake.com/lift?id=sc_cat_item&sys_id=de9fc362db7dd4102f1c9eb6db9619ed) ticket to request access.

**3. 🧑‍🔧 Incorrect User Role Usage**
  - **Issue:** Code fails due to role/permission issues
  - **Tip:** Follow the guide for when to use:
    - `mlops_user` (SageMaker programmatic access)
    - Your SE Snowflake account (Snowsight setup and model monitoring)

**4. ⚙️ Private Key or PAT Setup Fails**
  - **Error:** `"New public key rejected by current policy"` or `"Invalid public key"`
  - **Cause:** Account's authentication policy restricts key-based access
  - **Fix:**
    - **Regenerate the key pair** using the openssl commands above (#1 FAQ Tip).
    - Ensure your public key is correctly updated in your Snowflake user profile
    - Upload the private key file into SageMaker and update your `connections.toml`
    - Double-check that the private key **starts and ends with the required headers**, and avoid using rich-text editors that could introduce formatting problems

**5. 🔧 SNOWFLAKE_ACCOUNT Underscore-to-Hyphen Requirement**
  - **Issue:** `SSL: CERTIFICATE_VERIFY_FAILED` when connecting to Snowflake from SageMaker
  - **Cause:** the value for `SNOWFLAKE_ACCOUNT` contains an underscore, which is not allowed in the host URL used for connection
  - **Fix:**
    - Replace underscores (`_`) with hyphens (`-`) in your `SNOWFLAKE_ACCOUNT` value.
    - Example:
    - ❌ `SFSEEUROPE-US_DEMO603`
    - ✅ `SFSEEUROPE-US-DEMO603`
    - This resolves SSL errors caused by malformed domain names in HTTPS-based Snowflake connections.

**6. 🛠 Placeholder (XXX) Values Not Replaced**
  - **Issue:** Cells fail to execute or generate unexpected results
  - **Fix:** Use the provided **XXX Replacement Guide** to complete all fields such as:
    - `X_train`, `model_name`, `sample_input_data`, `PREDICTED_SCORE`, etc.
    - **Tip:** Use `search (XXX)` in your notebook to find all incomplete replacements.

**7. 📉 Model Registration or Explainability Errors**
  - **Error:** `"Explainability for this model is not supported"` or ValueError on registration
  - **Cause:** Incorrect input format or explainability limitations in current model setup
  - **Workaround:** Disable explainability temporarily and retry model registration.

**8. 🧪 DORA Completion Not Showing**
  - **Issue:** DORA steps (e.g., `SEAI50`, SEA51`, `SEAI52`, `SEAI53`, `SEAI54`) don’t appear as completed in the Tableau dashboard, even after finishing the lab
  - **Cause:** You may have skipped the grading and greeting function cells (often near the end of each notebook phase). Your Snowflake account may be missing a greeting assignment in the SE_GREETER function, which is required for linking your submissions to the dashboard.
  - **Fix:** ✅ **Ensure you check the completion box** at the end of each phase:
    - Look for a grading section in the notebook (e.g., RegisterRetrainedModel) that asks you to check a box or run a grading function **— this is required to trigger DORA tracking**
    - If you did not check the box or skipped that cell, **go back and complete it**
    - **🔁 Still not showing up?**
      - Double-check that your greeting function ran correctly, with mixed case for your first and last name:
      
      ```sql
      SELECT util_db.public.se_greeting('your.email@snowflake.com', 'First', '', 'Last');
      ```
      - Re-run the grading function for any missing step:
      ```sql
      SELECT util_db.public.se_grader('SEAI52', TRUE, 1, 1, 'Phase 1 complete');
      ```
      - Validate that your `util_db` has both the SE_GREETER and SE_GRADER external functions
      - 📝 Note: Even if your completion doesn't appear in Tableau, all submissions are still logged in Snowhouse and can be validated later if needed.

**9. 🧪 SageMaker JupyterLab Space Setup & Image Compatibility**
  - **Issue:** Users encounter unexpected errors (e.g., missing libraries, broken imports) when running notebooks
  - **Cause:** Selecting the **wrong JupyterLab image** (e.g., defaulting to 3.0.0 or another incompatible version). Confusion between **creating a new Domain** vs. **a JupyterLab space**. You're creating a **JupyterLab space**, not a new SageMaker Domain. The Domain was pre-created by SE-Sandbox admins. Only **JupyterLab spaces** incur cost while running — you’re safe to stop/restart without deleting anything.
  - **Required image:** `SageMaker Distribution 2.4.2`❗Other versions (e.g., `3.0.0`) will cause missing module errors.
  - **Fix:**
    - From **SageMaker Studio**, click **+ Create JupyterLab space**
      - Name it like `college-of-ai-yourname`
      - Select **Private space** (not a new Domain)
    - After creation, **switch image** to `SageMaker Distribution 2.4.2`
      - Errors like ModuleNotFoundError or broken ML commands will occur if using the wrong image
      - If resuming later, restart your space and **reconfirm image = 2.4.2**
    - ✅ Files persist between sessions — only running spaces incur cost

**10. 📎 Can’t Copy from Docs (Security Restriction)**
  - **Issue:** Unable to copy/paste from Google Docs
  - **Fix:** Use the official GitHub repository or PDFs distributed by the enablement team.

---
<a name="cleanup--cost-stewardship-procedures"></a>
## 🧹 Cleanup & Cost-Stewardship Procedures

🗑 **Cleanup Instructions:**
- **Phase 1:** Immediately shut down and delete your SageMaker instance through AWS Console:
  - Navigate to SageMaker > JupyterLab Spaces.
  - Stop and delete your workspace.
- **Phases 2 & 3:** Run the provided cleanup scripts at the end of each notebook

---
<a name="links-to-internal-resources--helpful-documents"></a>
## 🔗 Links to Internal Resources & Helpful Documents

💡 *For a deeper dive, check out the [50-minute PM-led presentation from the Delivery (PS) Enablement series](https://snowflake.sharepoint.com/sites/PSEnablementHub/SitePages/Delivery-Enablement-Series.aspx?OR=Teams-HL&CT=1707249903650&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiIyOC8yMzAxMDUwNzEwMCJ9). It explores Notebooks + Container Runtime in detail, including SPCS internals and practical examples. Highly recommended as a supplement to this HOL.*

### Helpful Documents:
- 📘 [Snowflake Model Registry Documentation](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/overview)
- ⚡ [Snowflake Model Registry Examples & Quickstarts](https://quickstarts.snowflake.com/guide/ml_model_registry/)
- 🚀 [Recommended End-to-End ML in Snowflake Quickstart](https://quickstarts.snowflake.com/guide/end_to_end_ml_in_snowflake/index.html?index=..%2F..index#0)
- 🔍 [Snowflake ML Observability Documentation](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/model-observability)
- 🧭 [Getting Started with ML Observability in Snowflake](https://quickstarts.snowflake.com/guide/ml_model_observability/)
- 🧮 [Inference in Snowflake Warehouses Documentation](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/warehouse)
- ❓ [How to do ML in Snowflake – FAQs](https://docs.snowflake.com/en/developer-guide/snowflake-ml/faqs)
- 📚 [Snowflake ML Reference API (v1.8)](https://docs.snowflake.com/en/developer-guide/snowflake-ml/reference)

---
<a name="author--support"></a>
## 👤 Author & Support

**Lab created by:** [Diana Shaw] – [Solution Engineering Enablement Principal]  
**Created on:** [May 2025] | **Last updated:** [July 2025]

💬 **Need Help or Have Feedback?**  
- Slack Channel: [#college-ai](https://snowflake.enterprise.slack.com/archives/C06QEKKADN1)
- Slack DM: [@diana.shaw](https://snowflake.enterprise.slack.com/team/U02DPRSJ2QL)
- Email: [diana.shaw@snowflake.com](mailto:diana.shaw@snowflake.com)

🌟 *We greatly value your feedback to continuously improve our HOL experiences!*
