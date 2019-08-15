Grab the variables

```
gcloud organizations list
gcloud beta billing accounts list
gcloud projects list 
```

## set the variables
```
export PROVISIONER_VAR_org_id=
export PROVISIONER_VAR_billing_account=
export PROVISIONER_ADMIN=${USER}-provisioner-admin
export PROVISIONER_CREDS=~/.config/gcloud/${USER}-provisioner-admin.json
```



##  enable the apis
```
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable cloudbilling.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable serviceusage.googleapis.com
```

## grant permissions to the service accounts
```
gcloud iam service-accounts create provisioner --display-name "provisioner admin account"
gcloud iam service-accounts keys create ${PROVISIONER_CREDS} --iam-account provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com
gcloud projects add-iam-policy-binding ${PROVISIONER_PROJECT} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/viewer
gcloud projects add-iam-policy-binding ${PROVISIONER_PROJECT} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/storage.admin
```

## enable apis
```
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable cloudbilling.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable serviceusage.googleapis.com
```


## map the iam policy 
```
gcloud organizations add-iam-policy-binding ${PROVISIONER_VAR_org_id} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/resourcemanager.projectCreator
gcloud organizations add-iam-policy-binding ${PROVISIONER_VAR_org_id} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/billing.user
```

## Create terraform storage bucket
```
gsutil mb -p ${PROVISIONER_PROJECT} gs://${PROVISIONER_PROJECT}

cat > backend.tf << EOF
terraform {
 backend "gcs" {
   bucket  = "${PROVISIONER_PROJECT}"
   prefix  = "terraform/state"
 }
}
EOF

gsutil versioning set on gs://${PROVISIONER_PROJECT}
```

## Export credentials
```
export GOOGLE_APPLICATION_CREDENTIALS=${PROVISIONER_CREDS}
export GOOGLE_PROJECT=${PROVISIONER_PROJECT}
```

## Terraform Project
```
variable "project_name" {}
variable "billing_account" {}
variable "org_id" {}
variable "region" {}

provider "google" {
 region = "${var.region}"
}

resource "random_id" "id" {
 byte_length = 4
 prefix      = "${var.project_name}-"
}

resource "google_project" "project" {
 name            = "${var.project_name}"
 project_id      = "${random_id.id.hex}"
 billing_account = "${var.billing_account}"
 org_id          = "${var.org_id}"
}

resource "google_project_services" "project" {
 project = "${google_project.project.project_id}"
 services = [
   "compute.googleapis.com"
 ]
}

output "project_id" {
 value = "${google_project.project.project_id}"
}
EOF 
```




