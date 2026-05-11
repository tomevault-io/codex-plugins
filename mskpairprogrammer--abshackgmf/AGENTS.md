<!-- Use this file to provide workspace-specific custom instructions to Copilot. -->
- [x] Verify that the copilot-instructions.md file in the .github directory is created.

- [x] Clone GitHub Repository
	<!-- Cloned successfully from https://github.com/DavidAwe00/ABSHackGMF -->
	<!-- Project Type: Python Streamlit application with AWS integration (DynamoDB, Bedrock) -->

- [x] Review Project Structure
	<!-- Python project with Streamlit dashboard for Auto Loan Metrics Analysis -->
	<!-- Uses AWS services: DynamoDB, Bedrock, S3 -->
	<!-- Main file: app.py, Dependencies: requirements.txt -->

- [x] Install Dependencies
	<!-- Install Python packages from requirements.txt -->

- [x] Configure AWS Credentials
	<!-- Set up .env file with AWS credentials and configuration -->

- [x] Run the Application
	<!-- Start the Streamlit dashboard -->

- [x] Multi-Source Architecture Implementation
	<!-- Created agent orchestrator for multiple data sources -->
	<!-- Implemented Form424HAgent and Form10DAgent -->
	<!-- Configured 3 S3 buckets: abs-ee, 424h-prospectus, 10-d -->

## Project Information
- **Language**: Python 3.8+
- **Framework**: Streamlit
- **AWS Services**: DynamoDB, Bedrock, S3
- **Purpose**: Auto Loan Metrics Extraction and Analysis Dashboard
- **Architecture**: Multi-agent system with orchestrator pattern

## Data Sources
1. **ABS-EE** (S3_BUCKET_NAME1): Auto loan securitization filings
2. **Form 424H** (S3_BUCKET_NAME2): Prospectus supplements
3. **Form 10-D** (S3_BUCKET_NAME3): Distribution reports

## Agent Architecture
- **AgentOrchestrator**: Routes queries to appropriate agents
- **VisualizationAgent**: Handles ABS-EE auto loan data (fully implemented)
- **Form424HAgent**: Handles 424H prospectus data (framework ready)
- **Form10DAgent**: Handles 10-D distribution data (framework ready)

---
> Source: [mskpairprogrammer/ABSHackGMF](https://github.com/mskpairprogrammer/ABSHackGMF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-10 -->
