# Phase 1: Model Development & SageMaker Integration - Setup

This guide walks you through setting up your Snowflake environment and first-time access to Amazon SageMaker Studio for the **Model Development & SageMaker Integration** phase.  Replace items in **<angle brackets>** to adapt the template for your own hands-on lab (HOL).

🔽 Jump to:
- [🛠️ Snowflake Setup](#snowflake-setup)
- [🛠️ SageMaker Setup](#sagemaker-setup)
- [📂 Foundational Knowledge](#foundation-knowledge)
- [➡️ Next Steps](#next-steps)
- [⚠️ Troubleshooting](#troubleshooting)

---
<a name="snowflake-setup"></a>
## Step 1.1: Snowflake Setup
In this HOL, you'll configure a secure Snowflake environment for MLOps workflows with SageMaker.

This lab uses a **dedicated Snowflake Service User (`mlops_user`)** for secure, automated SageMaker access—avoiding changes to your SE demo environment or network policies.

⚠️ [**Programmatic Access Tokens (PATs)**](https://docs.snowflake.com/en/user-guide/programmatic-access-tokens) **are GA, but their usage is still limited.**
While Snowflake now supports PATs, they are currently **only supported in REST API-based workflows**, such as:
- Calling Cortex services (e.g., Analyst, Complete, Agents, Document AI)
- External integrations like Slack bots or third-party applications using Snowflake’s REST APIs

🔐 Use [**key-pair (JWT) authentication**](https://docs.snowflake.com/en/user-guide/key-pair-auth-troubleshooting) for Snowpark and Python-based programmatic access to Snowflake.

### 1.1 Snowflake Environment Setup
1. Log in to your Snowflake account
2.  In **Snowsight** select **Projects ➜ Worksheets**
3. Click ➕ **New Worksheet**.  
4. Set your role to **ACCOUNTADMIN** using the header dropdown.
5. Copy–paste the SQL below and press ▸ Run.
   
```sql
-- Use elevated privileges
USE ROLE ACCOUNTADMIN;

-- Create a dedicated role, service user, database, and warehouse
CREATE OR REPLACE ROLE aicollege;

CREATE OR REPLACE USER mlops_user
  TYPE = SERVICE 
  DEFAULT_ROLE = aicollege 
  COMMENT = 'Service user for MLOps HOL';

-- Grant role to service user and your standard user
GRANT ROLE aicollege TO USER mlops_user;
GRANT ROLE aicollege TO USER <your_standard_user>;

-- Create database and warehouse
CREATE DATABASE IF NOT EXISTS aicollege;

CREATE WAREHOUSE IF NOT EXISTS aicollege
  WITH WAREHOUSE_SIZE = 'XSMALL'
  AUTO_SUSPEND = 300;

-- Grant required permissions
GRANT USAGE, OPERATE ON WAREHOUSE aicollege TO ROLE aicollege;
GRANT ALL ON DATABASE aicollege TO ROLE aicollege;
GRANT ALL ON SCHEMA aicollege.public TO ROLE aicollege;
GRANT CREATE STAGE ON SCHEMA aicollege.public TO ROLE aicollege;
GRANT SELECT ON FUTURE TABLES IN SCHEMA aicollege.public TO ROLE aicollege;
GRANT SELECT ON FUTURE VIEWS IN SCHEMA aicollege.public TO ROLE aicollege;

-- Create a staging area for uploads
CREATE STAGE IF NOT EXISTS aicollege.public.setup;
GRANT READ ON STAGE aicollege.public.setup TO ROLE aicollege;
```

📌 Note: Using `SYSADMIN` as your default role can cause issues when running Snowpark, Cortex, or Snowflake ML functions. These features expect scoped privileges tied to the current active role and may fail if implicit `SYSADMIN` permissions interfere with access control or external integrations.

### 1.1.1 Recommended Code: Set Default Role to AICOLLEGE
```sql
ALTER USER mlops_user SET DEFAULT_ROLE = AICOLLEGE;
```

### 1.1.2 Upload Your Private Key for `mlops_user`
To securely connect SageMaker to Snowflake, you'll set up key-pair authentication.
1. Open your terminal (Mac/Linux) or Git Bash (Windows).
2. Run the following commands to generate a private and public key:
```bash
openssl genrsa -out rsa_private_key.pem 2048
openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
```
3. Open rsa_public_key.pem file in a text editor. Copy your public key.
4. In Snowsight, run the following SQL (replace placeholder):
```sql
ALTER USER mlops_user SET RSA_PUBLIC_KEY='
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAA...
-----END PUBLIC KEY-----';
```
5. Confirm the key was set:
```sql
DESC USER mlops_user;
```

### 1.1.3 Upload Files & Load Data
1. Use AICOLLEGE Role
2. Navigate to Data >> Databases >> AICOLLEGE >> PUBLIC >> Stages >> SETUP
3. Click + Files and choose these two files
   - [Mortgage_Data.csv](/data/Mortgage_Data.csv) and
   - [MLObservabilityWorkflow.jpg](/data/NewTrainingData.csv) files
4. Click Upload
5. Return to your SQL Worksheet
6. Copy and run this code to create/load the required data 
```sql
-- Ensure you're using the correct context
USE ROLE aicollege;
USE DATABASE aicollege;
USE SCHEMA public;
USE WAREHOUSE aicollege;

-- Create a target table for inference data
CREATE OR REPLACE TABLE InferenceMortgageData (
    WEEK_START_DATE TIMESTAMP_NTZ,
    WEEK NUMBER(38, 0),
    LOAN_ID NUMBER(38, 0),
    TS VARCHAR,
    LOAN_TYPE_NAME VARCHAR,
    LOAN_PURPOSE_NAME VARCHAR,
    APPLICANT_INCOME_000S NUMBER(38, 1),
    LOAN_AMOUNT_000S NUMBER(38, 0),
    COUNTY_NAME VARCHAR,
    MORTGAGERESPONSE NUMBER(38, 0)
);

-- Define file format for CSV import
CREATE OR REPLACE FILE FORMAT aicollege.public.mlops
    TYPE = CSV
    SKIP_HEADER = 1
    FIELD_DELIMITER = ','
    TRIM_SPACE = TRUE
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    REPLACE_INVALID_CHARACTERS = TRUE
    ERROR_ON_COLUMN_COUNT_MISMATCH = FALSE;

-- Load the CSV file into the table
COPY INTO InferenceMortgageData
FROM @aicollege.public.setup/Mortgage_Data.csv
FILE_FORMAT = aicollege.public.mlops
ON_ERROR = ABORT_STATEMENT;
```

### 1.1.4 Integrations & Permissions Setup
1. Use AICOLLEGE Role
2. Copy and run this code to set necessary permissions
```sql
USE ROLE ACCOUNTADMIN;

-- Create an Email integration to receive notifications from Snowflake alerts 
CREATE OR REPLACE NOTIFICATION INTEGRATION ML_ALERTS
  TYPE = EMAIL
  ENABLED = TRUE
  ALLOWED_RECIPIENTS = ('<snowflake email>');

GRANT USAGE ON INTEGRATION ML_ALERTS TO ROLE aicollege;
GRANT EXECUTE ALERT ON ACCOUNT TO ROLE AICOLLEGE;
GRANT CREATE DYNAMIC TABLE ON SCHEMA AICOLLEGE.PUBLIC TO ROLE AICOLLEGE;

-- Grant usage on existing DORA integration and utility database
GRANT USAGE ON INTEGRATION dora_api_integration TO ROLE aicollege;
GRANT USAGE ON DATABASE util_db TO ROLE aicollege;
GRANT USAGE ON SCHEMA util_db.public TO ROLE aicollege;

-- Grant usage on DORA external functions
GRANT USAGE ON FUNCTION util_db.public.se_grader(VARCHAR,BOOLEAN,INTEGER,INTEGER,VARCHAR) TO ROLE aicollege;
GRANT USAGE ON FUNCTION util_db.public.se_greeting(VARCHAR,VARCHAR,VARCHAR,VARCHAR) TO ROLE aicollege;

-- -- SageMaker Network Policy Configuration (to be run once you get your SageMaker IP address after running a cell in SageMaker)
-- Create the network policy
-- CREATE NETWORK POLICY ALLOW_SAGEMAKER
--   ALLOWED_IP_LIST = ('<YOUR_SAGEMAKER_IP>')
--   COMMENT = 'Restrict access to SageMaker IPs for MLOps HOL';
--   -- Assign the policy to the service user only (NOT to the full account)
-- ALTER USER mlops_user SET NETWORK_POLICY = ALLOW_SAGEMAKER;
-- -- Confirm it's set correctly
-- DESC USER mlops_user;
```

**🚧 User Access Guidelines: When to use `mlops_user` vs Your SE Account User**
This lab uses two users to follow security best practices and avoid network policy conflicts.
Here’s when to use each:

**👤 Your Standard SE Account User:**
Use in Snowsight or Snowflake Notebooks for:
- Phase 1 - Step 1.1: Environment setup (roles, databases, warehouse, data loading)
- Phase 2 - Step 2.2: Review Model Monitoring with Snowflake ML Observability
- Phase 3 - Step 3.2: End-to-end Model Retraining in Snowflake ML

**⚙️ The `mlops_user` (SageMaker Only):**
Use `mlops_user` in the SageMaker environment for:
- Phase 1 - Step 1.3: Secure, programmatic connection from SageMaker to Snowflake for model operations. (Configured in connections.toml — avoids SE account network policy risks)
🚫 There is no need to log into Snowsight with `mlops_user`.

---
<a name="sagemaker-setup"></a>
## 1.2 AWS SageMaker Access
This guide walks you through using a pre-provisioned for you in the [AWS SE-Sandbox](https://us-west-2.console.aws.amazon.com/console/home?region=us-west-2), *not the SE-CAPSTONE-SANDBOX*. If you do not have access, please [complete a LIFT ticket](https://lift.snowflake.com/lift?id=sc_cat_item&sys_id=de9fc362db7dd4102f1c9eb6db9619ed) to request it. Once access is granted, proceed with the following steps.

### 1.2.1 Log into AWS SE-Sandbox
1. Log in to [SnowBiz Okta](https://snowbiz.okta.com/app/UserHome)
2. Navigate to AWS SE-Sandbox application
3. Click on SE-Sandbox and select the Contributor role
4. Select the US West (Oregon) region from the top-right dropdown menu
<br><img src="/images/sagemaker/SE_Sandbox.jpg" alt="SE Sandbox" width="650"/>

**🚨 Important: SageMaker JupyterLab Usage Policy**
Before you begin setting up your SageMaker environment, please be aware:

**➡️ SageMaker JupyterLab spaces are expensive** if left running when not in active use. This is a shared SE Sandbox resource, and excessive idle compute costs put our continued access at risk.

**🔹 Your Responsibility:**
- Always **STOP** your JupyterLab space when you're not actively working.
- Once you’ve completed the lab, **DELETE** your space to free up resources.

### 1.2.2 Navigate to Amazon SageMaker Studio
1. In the AWS Management Console, search for "SageMaker" and select it
2. In the SageMaker dashboard, click on Studio in the left navigation panel
3. Click on Launch SageMaker Studio
<br><img src="/images/sagemaker/SageMaker.jpg" alt="SageMaker Studio" width="650"/>
4. Once the console loads, click the orange button Open Studio
<br><img src="/images/sagemaker/Open_Studio.jpg" alt="Open Studio" width="650"/>

### 1.2.3 Create JupyterLab Instance
1. In the Studio launcher, select JupyterLab spaces
<br><img src="/images/sagemaker/JupyterLab.jpg" alt="JupyterLab space" width="650"/>
2. Click + Create JupyterLab space
3. Name your space something like college-of-ai-<your name>
4. For "Environment", select **SageMaker Distribution 2.4.2**
5. For "Instance type", select ml.t3.medium
6. Click Create space
<br><img src="/images/sagemaker/SageMakerDistribution.jpg" alt="SageMaker Distribution" width="650"/>
7. Once the space is ready, click Open JupyterLab
<br><img src="/images/sagemaker/Stop_Delete.jpg" alt="Open JupyterLab" width="650"/>
   
**IMPORTANT: Always STOP your JupyterLab space when not actively working and DELETE it after completing the lab to avoid unnecessary charges.**

**CAUTION:** After stopping and restarting the space, SageMaker may default to the **latest image version (e.g. 3.0.0)**. Be sure to manually switch the image back to **SageMaker Distribution 2.4.2** before continuing. This ensures compatibility with the lab steps.

### 1.2.4 Upload Required Files
1. In JupyterLab, click the upload icon in the left sidebar
2. Upload the following files:
    - [connections.toml](/config/connections.toml) (update with your Snowflake credentials)
    - rsa_private_key.pem (your private key file)
    - [College-of-AI-MLOPsExerciseNotebook.ipynb](/notebooks/College-of-AI-MLOPsExerciseNotebook.ipynb) (provided notebook)
    - [Mortgage_Data.csv](https://drive.google.com/file/d/1fsL0y5HRNcswzgvTwgQJK9dfL6bCznGg/view?usp=sharing) (if needed for local processing)
3. Verify the uploaded files appear in your JupyterLab file browser

### 1.2.5 Configure Snowflake Connection
1. Open the [connections.toml](/config/connections.toml) file in JupyterLab
<br><img src="/images/sagemaker/ConnectionsTOML.jpg" alt="Connections file" width="650"/>
2. Update the file with your Snowflake connection details:
```toml
[connections.snowflake]
account = "your_account_identifier"
user = "mlops_user"
role = "aicollege"
warehouse = "aicollege"
database = "aicollege"
schema = "public"
private_key_path = "rsa_private_key.pem"
```
3. Save the file

---
<a name="foundation-knowledge"></a>
## Understanding Binary Classification Models

Before diving into building a model, let’s quickly walk through what we’re solving and how we’ll prepare the data.

### 🏦 Scenario: Mortgage Approval Prediction

You’re working with **historical mortgage application data**.  
Each row represents an individual application, and your task is to **predict whether the mortgage was approved or denied**:

| Label | Meaning |
|-------|---------|
| **`MORTGAGERESPONSE = 1`** | Approved |
| **`MORTGAGERESPONSE = 0`** | Denied |

This is a classic **binary classification** problem: only two possible outcomes — **approved or denied**.

### 📊 Why Logistic-Style (Tree-Based) Models?

We’ll train a **binary classifier** using **`XGBoostClassifier`**, a tree-based model widely used in production. XGBoost is great for binary decisions because it:
- Output a **probability between 0 and 1**
- Allow you to **set thresholds** (e.g., approve if probability > 0.5)
- Capture non-linear relationships between input feature data and output classification
- Works well for use cases like:
   - Fraud detection
   - Credit scoring
   - Loan or insurance approvals
   - Customer churn prediction

### 🔍 Features We’ll Use

| Type | Example Features |
|------|------------------|
| **Numeric** | `APPLICANT_INCOME_000S`, `LOAN_AMOUNT_000S` |
| **Categorical** | `LOAN_TYPE_NAME`, `LOAN_PURPOSE_NAME`, `COUNTY_NAME` |

These features will be cleaned and transformed before model training.

### 💡 What Is One-Hot Encoding?

ML models like XGBoost **can’t interpret text directly**. That means we need to convert categorical columns like "`FHA`" or "`Conventional`" into a numeric format.
That’s where one-hot encoding comes in. 
Let’s say your original data looks like this:

|`LOAN_TYPE_NAME`|
|----|
|Conventional|
|FHA-insured|
|VA-guaranteed|

One-hot encoding transforms it into **separate binary columns**, like so:

| LOAN_TYPE_NAME_Conventional | LOAN_TYPE_NAME_FHA | LOAN_TYPE_NAME_VA |
|----------------------------:|-------------------:|------------------:|
| 1 | 0 | 0 |
| 0 | 1 | 0 |
| 0 | 0 | 1 |

This prevents the model from treating category values as ranked numbers (e.g., “FHA” > “VA”).

> **Tip:** Snowflake ML’s built-in `OneHotEncoder` can do this directly on Snowpark DataFrames—ideal for production pipelines. See the [Snowflake ML OneHotEncoder docs](https://docs.snowflake.com/en/user-guide/snowpark-ml).

### 🔬 Evaluating Your Model

After training, evaluate performance on **unseen validation data**.

#### 📊 Classification Report

You’ll generate a report with these metrics:

| Metric | What it tells you | Why it matters |
|--------|------------------|----------------|
| **Precision** | Of all predicted approvals, how many were actually approved? | High precision ⇒ few risky loans get approved. |
| **Recall** | Of all actual approvals, how many did the model catch? | High recall ⇒ few good applicants are missed. |
| **F1-Score** | Harmonic mean of precision & recall | Balances false positives and false negatives. |
| **Support** | Number of actual instances per class | Helps interpret the other metrics. |

#### 🧮 Confusion Matrix

A grid showing correct vs. incorrect predictions.

| | **Predicted: Denied** | **Predicted: Approved** |
|---|---|---|
| **Actual: Denied** | ✅ **True Negatives** (14 219) | ❌ **False Positives** (371) |
| **Actual: Approved** | ❌ **False Negatives** (211) | ✅ **True Positives** (48 348) |

* **True Negatives** – correctly denied risky apps  
* **False Positives** – incorrectly approved risky apps  
* **False Negatives** – missed good applicants  
* **True Positives** – correctly approved good applicants  

These numbers reveal whether the model leans toward over-approving or over-denying.

### 🧳 Snowflake Model Registry + Batch Inference

#### ✅ Model Registry

* Register models by **name & version**  
* Track dependencies / schemas  
* Call via `model.run()` (Python) or `PREDICT_PROBA()` (SQL)  
* Manage version promotion to production  

#### 📈 Batch Inference

Enterprise workflows often score data in bulk:

* Nightly loan-approval scoring  
* Weekly churn predictions  
* End-of-day fraud risk analysis  

In this HOL you will:

| Scoring Type | Purpose |
|--------------|---------|
| **Weekly batch** (Weeks 1–5) | Simulate a production job processing new records each week |
| **Full-table scoring** | Back-test, prep ML Observability, feed dashboards |

Both predicted labels and **prediction confidence** (`PREDICTED_SCORE`) are stored in Snowflake.

#### Why Confidence Scores Matter

* Visualize score distributions by segment  
* Detect data / concept drift  
* Track precision, recall, F1, ROC AUC trends  
* Power dashboards & alerting systems  

---
<a name="next-steps"></a>
### 1.2.6 Next Steps
Now that your Snowflake environment is set up and you have access to SageMaker, you're ready to proceed with [**Phase 1: Model Development in SageMaker - Initial Model Registration**](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/lab_instructions/phase1_model_dev.md).

## 📘 What You’ll Do in the SageMaker Notebook

1. **Load** historical data from S3.  
2. **Pre-process** (drop nulls, one-hot encode).  
3. **Train** an XGBoost binary classifier.  
4. **Evaluate** with precision/recall & confusion matrix.  
5. **Save** the model locally (`.pkl`).  
6. **Register** it in the Snowflake Model Registry (`log_model()`).  
7. **Run batch inference**  
   * Week-by-week (Weeks 1–5)  
   * Full table  
8. **Write predictions** back to Snowflake.  
9. Complete DORA evaluations **SEAI50** & **SEAI51** to confirm registration and inference.

> ✅ No APIs, no endpoints—just batch ML using SQL + Python inside Snowflake.

You’re now ready to begin developing a predictive model in **SageMaker**!

---
<a name="troubleshooting"></a>
## Troubleshooting
### Common Snowflake Issues
- **Permission errors:** Verify that the aicollege role has all necessary privileges
- **Connection issues:** Ensure your account identifier is correct and includes region if needed (e.g. `xy12345.us-east-1`)
  - *Remember that `account` cannot contain underscores. Change to a hyphen. For instance, SFSEEUROPE-US_DEMO603 needs to be SFSEEUROPE-US-DEMO603.*
- **Key-pair authentication:** Make sure the private key matches the public key registered with the user

### Common SageMaker Issues
- **JupyterLab fails to start:** Try selecting a different instance type or refreshing the page
- **Kernel disconnections:** Restart the kernel or refresh the page
