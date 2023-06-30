reffer
https://cloud.google.com/architecture/manage-just-in-time-privileged-access-to-project#organization

step1 1
```shell
gcloud config set project ethanhanjoonix-proj1
SERVICE_ACCOUNT=$(gcloud iam service-accounts create jitaccess --display-name "Just-In-Time Access" --format "value(email)")
SCOPE_ID=265966344803
gcloud organizations add-iam-policy-binding $SCOPE_ID \
    --member "serviceAccount:$SERVICE_ACCOUNT" \
    --role "roles/iam.securityAdmin" \
    --condition None

```

step 2
add service account as group reader

step 3  cloud run
```shell
gcloud config set run/region asia-southeast1
gcloud compute backend-services create jitaccess-backend \
  --load-balancing-scheme=EXTERNAL \
  --global
git clone https://github.com/GoogleCloudPlatform/jit-access.git
cd jit-access/sources
git checkout latest


PROJECT_ID=$(gcloud config get-value core/project)
REGION=asia-southeast1
REPO_NAME=jit-repo
SERVICE=jit-access
#docker build -t gcr.io/$PROJECT_ID/jitaccess:latest .
docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${SERVICE}:latest . 
gcloud artifacts repositories create ${REPO_NAME} \
    --repository-format="DOCKER" \
    --location=${REGION} \
    --description="DESCRIPTION" \
    --async
gcloud auth configure-docker asia-southeast1-docker.pkg.dev
docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${SERVICE}:latest


PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format 'value(projectNumber)')
REGION=$(gcloud config get-value run/region)
IAP_BACKEND_SERVICE_ID=$(gcloud compute backend-services describe jitaccess-backend --global --format 'value(id)')

cat << EOF > app.run.yaml

apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: jitaccess
  namespace: $PROJECT_NUMBER
  labels:
    cloud.googleapis.com/location: $REGION
spec:
  template:
    spec:
      serviceAccountName: $SERVICE_ACCOUNT
      containers:
      - image: ${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPO_NAME}/${_SERVICE}:latest
        env:
        - name: RESOURCE_SCOPE
          value: "$SCOPE_TYPE/$SCOPE_ID"
        - name: ELEVATION_DURATION
          value: "60"
        - name: JUSTIFICATION_HINT


gcloud run services replace app.yaml





gcloud pubsub topics create jit-access
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SERVICE_ACCOUNT" --role=roles/pubsub.publisher
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SERVICE_ACCOUNT" --role=roles/pubsub.subscriber

wget https://dlcdn.apache.org/maven/maven-3/3.9.2/binaries/apache-maven-3.9.2-bin.zip
unzip apache-maven-3.9.2-bin.zip
export PATH=$PATH:/home/user/apache-maven-3.9.2/bin
mvn -v
cd 



gcloud services enable secretmanager.googleapis.com --project=$PROJECT_ID

INTEGRATION_TEST_SA_NO_ACCESS="no-access"
INTEGRATION_TEST_SA_TEMP_ACCESS="temporary-access"
gcloud iam service-accounts create $INTEGRATION_TEST_SA_NO_ACCESS
gcloud iam service-accounts create $INTEGRATION_TEST_SA_TEMP_ACCESS

gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$INTEGRATION_TEST_SA_NO_ACCESS@$PROJECT_ID.iam.gserviceaccount.com" --role=roles/iam.serviceAccountUser
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$INTEGRATION_TEST_SA_TEMP_ACCESS@$PROJECT_ID.iam.gserviceaccount.com" --role=roles/iam.serviceAccountTokenCreator


mvn compile quarkus:test
mvn compile quarkus:dev



```