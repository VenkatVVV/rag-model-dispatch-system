# aletha-project
## Databricks RAG Model Router and Specialized ML Models 

This repository (or Databricks workspace setup) contains a Retrieval Augmented Generation (RAG) model designed to intelligently route user queries to specialized machine learning models or a general-purpose Large Language Model (LLM) based on the intent of the query.

#### 1. RAG Model Router (retrievalAugmentedGenerationModel notebook)
Purpose: Acts as a central dispatcher for various data science inquiries. It analyzes a user's natural language query and determines the most appropriate action:

- Executing a specific machine learning model notebook.

- Delegating to a general LLM for broader questions.

Functionality:

- Query Classification: Uses keyword matching and regular expressions to identify the domain of the user's request (e.g., "fraud," "credit risk," "customer segmentation").

- Notebook Routing: If a specific domain is identified, it dynamically executes the corresponding Databricks notebook containing the specialized ML model.

- LLM Integration: For general inquiries that don't match a specific ML model, it calls an external LLM (Gemini API) to generate a relevant response.

- Logging: Incorporates Python's logging module to track query classifications, notebook executions, LLM calls, and any errors for auditing and troubleshooting.

- Access Controls: Designed to work with Databricks Secrets for secure API key management and assumes appropriate notebook/cluster/table ACLs are in place for secure operation.

#### 2. Specialized Machine Learning Models
##### 2.1. Fraud Detection Model (fraudDetection notebook)
Purpose: To identify potentially fraudulent transactions based on transactional and customer behavioral patterns.

Functionality:

- Data Sources: Utilizes customers and transactions tables.

- Feature Engineering: Derives various features from the raw data, including:

  - Transaction amount, recipient, device type.

  - Customer demographics (age, income, credit score, DTI, LTV).

  - Aggregated transaction history per customer (total spending, average amount, number of transactions).

  - Time-windowed features (e.g., transaction count/sum in the last hour/day) to detect rapid, unusual activity.

  - Transaction hour of day to identify unusual timing.

- Labeling (Dummy): For training purposes, a synthetic is_fraudulent label is created based on predefined rules (e.g., high amount, unusual recipient/device, high frequency/sum in short periods, transactions at unusual hours). In a real-world scenario, this would be based on historical, confirmed fraud data.

- Machine Learning Model: Employs a Logistic Regression classifier (from Spark MLlib) to predict the likelihood of a transaction being fraudulent.

- Output: Returns a list of transaction records, ranked by their predicted probability of being fraudulent, allowing for targeted investigation.

##### 2.2. Credit Risk Regression Model (creditRiskScoreModel notebook)
Purpose: To assess the creditworthiness of customers by predicting their credit score, thereby ranking them from highest to lowest risk.

Functionality:

- Data Sources: Primarily uses the customers table, with aggregated features from the transactions table.

- Feature Engineering: Combines customer demographic and financial attributes with aggregated transaction history (e.g., total spending, average transaction amount, number of transactions, spending velocity).

- Target Variable: The credit_score column from the customers table serves as the target variable for the regression model.

- Machine Learning Model: Uses a Linear Regression model (from Spark MLlib) to predict a continuous credit score for each customer.

- Output: Generates a list of customers, ranked from highest risk (lowest predicted credit score) to lowest risk (highest predicted credit score).

##### 2.3. Customer Segmentation Model (customerSegmentationModel notebook)
Purpose: To categorize customers into distinct groups based on their income and spending habits for targeted marketing or strategic analysis.

Functionality:

- Data Sources: Utilizes the customers table for annual_income and derives Average Monthly Spending by aggregating amount from the transactions table.

- Feature Preparation:

  - Selects annual_income and the calculated Average Monthly Spending.

  - Applies VectorAssembler to combine these features.

  - Uses StandardScaler to normalize the features, ensuring both income and spending contribute equally to the clustering process.

- Machine Learning Model: Implements K-Means Clustering (from Spark MLlib) to group customers into 4 distinct categories.

- Cluster Interpretation: Calculates the average income and spending for each cluster to help interpret the meaning of each segment.

- Category Assignment: Assigns human-readable labels (e.g., "High Income High Spenders," "Low Income Low Spenders") to each cluster based on their interpreted characteristics.

- Output: Provides a list of customers with their assigned income-spending category.

#### Technologies Used
- Databricks: The unified analytics platform for running Spark workloads.

- Apache Spark (PySpark): For large-scale data processing, feature engineering, and ML model training.

- Spark MLlib: Spark's native machine learning library.

- Python: The primary programming language for all notebooks.

- Gemini API: For general inquiry responses within the RAG router.

### Setup and Usage
1. Workspace Setup: Ensure all three specialized ML model notebooks (fraudDetection, creditRiskScoreModel, customerSegmentationModel) are present in your Databricks workspace at the specified paths.

2. Database and Tables: Verify that your customers and transactions Delta tables exist in the your_database_name database (update this placeholder in the code).

3. LLM API Key: Create a Databricks Secret Scope (e.g., rag_secrets_scope) and store your Gemini API key under a secret key (e.g., gemini_api_key). Update the LLM_API_SECRET_SCOPE and LLM_API_SECRET_KEY variables in the retrievalAugmentedGenerationModel notebook.

4. Permissions: Grant appropriate RUN permissions to the databricks-rag-router notebook, and ensure it (or the user/service principal running it) has RUN permissions on the target ML model notebooks and SELECT permissions on the underlying Delta tables.

5. Execution: Run the retrievalAugmentedGenerationModel notebook. It will prompt you for a query.

### Important Notes
- spark.ml Whitelisting: While spark.ml is a core component of Apache Spark and generally available on Databricks, some specific environments (like certain serverless configurations or clusters with very strict security policies) might encounter Py4JSecurityException related to StringIndexer or other spark.ml components. This typically indicates a security restriction on underlying Java constructor calls. If you face this, ensure your Databricks Runtime is a standard, recent LTS version, and if the issue persists, consult your Databricks administrator or Databricks support.

- Gemini API Key: The RAG router's LLM integration requires a valid Gemini API key. This key should be securely stored in Databricks Secrets and retrieved programmatically as shown in the retrievalAugmentedGenerationModel notebook. You will need to obtain this key from Google Cloud Console or Google AI Studio if you don't already have one.

This README provides a comprehensive overview of your RAG system and its components.