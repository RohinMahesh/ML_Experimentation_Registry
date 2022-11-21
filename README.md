# ML Experimentation Tracking and Registry with MLflow

## Background

MLflow is an excellent open source library for model registry and experimentation tracking that enables version control and governance over your ML artifacts. MLflow is compatible with various leading MLOps frameworks and supports in Java, Python, R and Spark. In the last few years, MLflow has introduced Model Serving, which allows you to host your ML artifacts as REST endpoints.

MLflow contains the following services:
- Backend Store: stores all experiment metadata
- Artifacts Store: stores all ML artifacts
- Tracking Server: enables experiment tracking and ML artifact registry

Disclaimers:
- You will need a Policy in IAM that will allow you to perform the required actions for the different MLflow services above
- ML artifacts are never promoted directly to Production. All ML artifacts are promoted to Staging and manually promoted to Production based on testing and experiment information

## Backend Store

The Backend Store in this project is created be leveraging AWS RDS (PostgreSQL). In this project, we leverage the free tier options for our template. The following options are selected:

- Credential Settings
    - DB instance identifier: mlflow_db
    - Master username: mlflow
    - Auto generate a password
- Additional Configuration
    - Initial database name: mlflow_data
- Connectivity & security
    - Under Security -> VPC security groups:
        - Inbound rules:
            - Type: PostgreSQL
            - Protocol: TCP
            - Port range: 5432
            - Source: Custom
            - CIDR block: security group name created when EC2 instance was launched

## Artifacts Store

The Artifacts Store in this project is created by leveraging AWS S3. In this project, we create a bucket called "mlflow-artifact-store".

## Tracking Server

The Tracking Server in this project is created by leveraging AWS EC2. This component of MLflow assumes that both the Backend and Artifacts Store are functional and accessible by the Tracking Server. 

When creating an EC2 instance, the following components are required:
- Key Pair: a PEM file will be required to SSH into the Tracking Server from your experiment/code. This PEM file is selected in your EC2 instance.
- Inbound Rules: under Security Group, the following is modified from defaults:
    - Type: Custom TCP
    - Port range: 5000
    - Source: custom
    - CIDR blocks: 0.0.0.0/0

Allowing access from all the IPs (CIDR block) is not recommended in a production setting as this may be a security risk. 

After connecting to your instance, run the following from your console:

Update all libraries on the system:

```bash
sudo yum update
```

Install your MLflow dependencies:

```bash
pip3 install mlflow boto3 psycopg2-binary
```

Configure AWS credentials:

```bash
aws configure
```

Start your MLflow Tracking Server and make the following modifications:
- Replace DB_PASSWORD with the password you copied when clicking "View crediential details" upon creating your PostgreSQL database.
- Replace DB_ENDPOINT with the endpoint that you were provided when creating the PostgreSQL database.

```bash
mlflow server -h 0.0.0.0 -p 5000 –backend-store-uri postgresql://mlflow:DB_PASSWORD@DB_ENDPOINT:5432/mlflow_data –default-artifact-root s3://mlflow-artifact-store
```

You can now access your MLflow Tracking Server by using the endpoint under "Public IPv4 DNS" using port 5000. This endpoint should be used in your local experiments. Below is an example where "TrackingServerEndpoint" is your endpoint from "Public IPv4 DNS":

```bash
mlflow.set_tracking_uri(f"https://{TrackingServerEndpoint}:5000")
```
