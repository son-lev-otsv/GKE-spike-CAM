# Cloud SQL and Kubernetes Integration Guide - Step by Step Setup

## Step 1: Project Setup
```bash
# Set project
gcloud config set project YOUR_PROJECT_ID

# Login to gcloud
gcloud auth login
gcloud auth application-default login
```
Initialize and authenticate with Google Cloud.

## Step 2: Network Setup
```bash
# Create firewall rule for SQL connections
gcloud compute firewall-rules create allow-private-sql \
    --network=cam-demo-network \
    --direction=INGRESS \
    --priority=1000 \
    --source-ranges=10.0.16.0/20 \
    --action=ALLOW \
    --rules=tcp:5432,tcp:3306 \
    --target-tags=private-sql
```
Configure network security for database access.

## Step 3: Service Account Setup
```bash
# Create service account key
gcloud iam service-accounts keys create service-account.json \
    --iam-account=YOUR_SERVICE_ACCOUNT_EMAIL

# Link KSA (Kubernetes Service Account) with GSA (Google Service Account)
kubectl annotate serviceaccount YOUR_KSA_NAME \
    iam.gke.io/gcp-service-account=YOUR_GSA_EMAIL
```
Set up authentication between GKE and Cloud SQL.

## Step 4: Kubernetes Secret Configuration
```bash
# Create Cloud SQL instance credentials secret
kubectl create secret generic cloudsql-instance-credentials \
  --from-file=service_account.json=./service-account.json

# Create database credentials secret
kubectl create secret generic cloudsql-db-credentials \
  --from-literal=username=postgres \
  --from-literal=password=YOUR_PASSWORD \
  --from-literal=database=postgres
```
Store sensitive credentials securely in Kubernetes.

## Step 5: ConfigMap Setup
```bash
# Create application configuration
kubectl create configmap cam-demo-config \
  --from-literal=DB_HOST=127.0.0.1 \
  --from-literal=DB_PORT=5555 \
  --from-literal=INSTANCE_CONNECTION_NAME=PROJECT:REGION:INSTANCE \
  --from-literal=APP_PORT=80 \
  --from-literal=ENVIRONMENT=production \
  -o yaml --dry-run=client | kubectl replace -f -
```
Configure application environment variables.

## Step 6: Cloud SQL Proxy Setup
```bash
# Install Cloud SQL Proxy (local development)
brew install cloud-sql-proxy

# Start Cloud SQL Proxy
cloud-sql-proxy PROJECT_ID:REGION:INSTANCE --port=5432
```
Enable secure connection to Cloud SQL.

## Step 7: Testing and Verification
```bash
# Port forwarding
kubectl port-forward POD_NAME 5432:5432

# Test database connection
kubectl exec -it POD_NAME -- psql -h localhost -U postgres -d postgres
```
Verify database connectivity.

## Step 8: Context Management
```bash
# View current context
kubectl config current-context

# Switch context if needed
kubectl config use-context CONTEXT_NAME
```
Manage different Kubernetes cluster contexts.

## Important Configuration Values
```plaintext
Database Configuration:
- Version: PostgreSQL 16
- Region: us-central1 (Iowa)
- Availability: Multiple zones (highly available)
- Default Port: 5432
- Connection Name Format: PROJECT_ID:REGION:INSTANCE_NAME
```

## Security Notes:
- Always keep service account keys secure
- Use appropriate network security rules
- Follow least privilege principle for service accounts
- Regularly rotate credentials
- Use secrets for sensitive information
- Enable audit logging for database access

## Troubleshooting:
1. Verify service account permissions
2. Check network connectivity
3. Ensure correct context is selected
4. Verify secret and configmap creation
5. Check pod logs for connection issues

Remember to replace placeholder values (PROJECT_ID, REGION, INSTANCE, etc.) with your actual configuration values.