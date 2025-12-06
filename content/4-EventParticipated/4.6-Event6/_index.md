---
title: "Workshop: Data Science on AWS"
date: 2025-10-16
weight: 6
chapter: false
pre: " <b> 4.6. </b> "
---

# Summary Report: “Workshop: Data Science on AWS”

### Event Objectives

-   Explore the complete journey of building a modern Data Science system from theory to practice.
-   Understand the end-to-end Data Science Pipeline on AWS, from storage to processing and deployment.
-   Gain hands-on experience with real-world datasets (IMDb) and practical models (Sentiment Analysis).
-   Analyze the trade-offs between Cloud and On-premise infrastructures regarding cost and performance.

### Speakers

-   **Mr. Văn Hoàng Kha** – Cloud Solutions Architect, AWS Community Builder
-   **Mr. Bạch Doãn Vương** – Cloud DevOps Engineer, AWS Community Builder

### Key Highlights

#### 1. Cloud in Data Science & Pipeline Overview
-   **Importance of Cloud**: Discussed why modern data science relies on the cloud for scalability and integration, moving away from rigid on-premise constraints.
-   **The AWS Data Science Pipeline**:
    -   **Storage**: Using **Amazon S3** as the foundational data lake.
    -   **ETL/Processing**: Utilizing **AWS Glue** for serverless data integration.
    -   [cite_start]**Modeling**: Leveraging **Amazon SageMaker** as the central hub for building, training, and deploying models[cite: 212].
    -   [cite_start]The session overviewed the broad AWS AI/ML stack, which encompasses AI services, ML services, and infrastructure[cite: 201].

#### 2. Practical Demos
-   **Demo 1: Data Processing with AWS Glue**:
    -   **Scenario**: Processing and cleaning raw data from the **IMDb dataset**.
    -   **Technique**: Demonstrated how to handle feature engineering and data preparation efficiently. [cite_start]The workshop highlighted different approaches, ranging from low-code options like **SageMaker Canvas** [cite: 209] [cite_start]to code-first methods using **Numpy/Pandas**[cite: 210].
-   **Demo 2: Sentiment Analysis with SageMaker**:
    -   **Scenario**: Training and deploying a machine learning model to analyze text sentiment.
    -   [cite_start]**Workflow**: Showcased the "Train, Tune, Deploy" lifecycle within SageMaker Studio[cite: 212]. [cite_start]The session also touched upon "Bring Your Own Model" (BYOM) concepts, demonstrating flexibility with frameworks like TensorFlow and PyTorch[cite: 213].

#### 3. Strategic Discussions
-   **Cloud vs. On-Premise**: A deep dive into cost optimization and performance metrics. The discussion highlighted how cloud elasticity allows for experimenting with heavy workloads without the massive upfront capital expenditure of on-premise hardware.
-   **Mini-Project Guidance**: Introduction to a post-workshop project designed to reinforce the skills learned.

### Key Takeaways

#### Technical Workflow
-   **Unified Pipeline**: A robust data science workflow is not just about the code; it requires seamless integration between storage (S3), cleaning (Glue), and modeling (SageMaker).
-   [cite_start]**Tool Selection**: Understanding when to use managed services (like Amazon Comprehend or Textract [cite: 202, 203]) versus building custom models on SageMaker is crucial for efficiency.

#### Industry Application
-   **Real-world Context**: The transition from academic theory to industry application lies in automation and scalability.
-   **Cost Awareness**: Successful data projects must balance model accuracy with computational costs.

### Applying to Work

-   **Adopt AWS Glue**: Migrate local ETL scripts to AWS Glue for automated, serverless data cleaning on larger datasets.
-   **SageMaker Deployment**: Move experimental models from local Jupyter notebooks to SageMaker Studio to standardize the training and deployment process.
-   **Project Implementation**: Execute the suggested mini-project to solidify understanding of the IMDb processing workflow.

### Event Experience

The **“Data Science on AWS”** workshop was a bridge between university curriculum and enterprise reality.
-   **Direct Connection**: It connected academic knowledge with the technologies used by top global enterprises.
-   **Practical Insight**: Watching the **IMDb dataset** being cleaned and a **Sentiment Analysis** model being deployed live demystified the complexity of cloud-based AI.
-   **Expert Guidance**: The interaction with AWS Community Builders provided deep insights into the "Cloud vs. On-premise" debate, helping me understand the strategic value of cloud migration beyond just technical features.

#### Some event photos

_Add your event photos here_