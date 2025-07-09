# Phase 1: Model Development & SageMaker Integration - Setup

This guide walks you through setting up your Snowflake environment and first-time access to Amazon SageMaker Studio for the **Model Development & SageMaker Integration** phase.  Replace items in **<angle brackets>** to adapt the template for your own hands-on lab (HOL).

---

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

## 1.2 AWS SageMaker Access
This guide walks you through using a pre-provisioned for you in the [AWS SE-Sandbox](https://us-west-2.console.aws.amazon.com/console/home?region=us-west-2), *not the SE-CAPSTONE-SANDBOX*. If you do not have access, please [complete a LIFT ticket](https://lift.snowflake.com/lift?id=sc_cat_item&sys_id=de9fc362db7dd4102f1c9eb6db9619ed) to request it. Once access is granted, proceed with the following steps.

### 1.2.1 Log into AWS SE-Sandbox
1. Log in to [SnowBiz Okta](https://snowbiz.okta.com/app/UserHome)
2. Navigate to AWS SE-Sandbox application
3. Click on SE-Sandbox and select the Contributor role
4. Select the US West (Oregon) region from the top-right dropdown menu
html<br><img src="/images/sagemaker/SE_Sandbox.jpg" alt="SE Sandbox" width="650"/>

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
html<br><img src="/images/sagemaker/SageMaker.jpg" alt="SageMaker Studio" width="650"/>
4. Once the console loads, click the orange button Open Studio
html<br><img src="/images/sagemaker/Open_Studio.jpg" alt="Open Studio" width="650"/>

### 1.2.3 Create JupyterLab Instance
1. In the Studio launcher, select JupyterLab spaces
html<br><img src="/images/sagemaker/JupyterLab.jpg" alt="JupyterLab space" width="650"/>
2. Click + Create JupyterLab space
3. Name your space something like college-of-ai-<your name>
4. For "Environment", select **SageMaker Distribution 2.4.2**
5. For "Instance type", select ml.t3.medium
6. Click Create space
html<br><img src="/images/sagemaker/SageMakerDistribution.jpg" alt="SageMaker Distribution" width="650"/>
7. Once the space is ready, click Open JupyterLab
html<br><img src="/images/sagemaker/Stop_Delete.jpg" alt="Open JupyterLab" width="650"/>
   
**IMPORTANT: Always STOP your JupyterLab space when not actively working and DELETE it after completing the lab to avoid unnecessary charges.**

**CAUTION:** After stopping and restarting the space, SageMaker may default to the **latest image version (e.g. 3.0.0)**. Be sure to manually switch the image back to **SageMaker Distribution 2.4.2** before continuing. This ensures compatibility with the lab steps.

### 1.2.4 Upload Required Files
1. In JupyterLab, click the upload icon in the left sidebar
2. Upload the following files:
    - [connections.toml](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/config/connections.toml) (update with your Snowflake credentials)
    - rsa_private_key.pem (your private key file)
    - [College-of-AI-MLOPsExerciseNotebook.ipynb](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/notebooks/College-of-AI-MLOPsExerciseNotebook.ipynb) (provided notebook)
    - [Mortgage_Data.csv](https://drive.google.com/file/d/1fsL0y5HRNcswzgvTwgQJK9dfL6bCznGg/view?usp=sharing) (if needed for local processing)
3. Verify the uploaded files appear in your JupyterLab file browser

### 1.2.5 Configure Snowflake Connection
1. Open the [connections.toml](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/config/connections.toml) file in JupyterLab
html<br><img src="/images/sagemaker/ConnectionsTOML.jpg" alt="Connections TOML" width="650"/>
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



### 1.2.6 Next Steps
Now that your Snowflake environment is set up and you have access to SageMaker, you're ready to proceed with [**Phase 1: Model Development in SageMaker - Initial Model Registration**](https://github.com/sfc-gh-DShaw98/SageMaker-to-Snowflake-Batch-Inference-Lab/blob/main/lab_instructions/phase1_model_dev.md).

## Troubleshooting
### Common Snowflake Issues
- **Permission errors:** Verify that the aicollege role has all necessary privileges
- **Connection issues:** Ensure your account identifier is correct and includes region if needed (e.g. `xy12345.us-east-1`)
  - *Remember that `account` cannot contain underscores. Change to a hyphen. For instance, SFSEEUROPE-US_DEMO603 needs to be SFSEEUROPE-US-DEMO603.*
- **Key-pair authentication:** Make sure the private key matches the public key registered with the user

### Common SageMaker Issues
- **JupyterLab fails to start:** Try selecting a different instance type or refreshing the page
- **Kernel disconnections:** Restart the kernel or refresh the page
