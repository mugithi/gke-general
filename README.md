Grab the variables

```
gcloud organizations list
gcloud beta billing accounts list
gcloud projects list 
```

## set the variables
```
export PROVISIONER_PROJECT=
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

## Export Terraform Variables

```
export TF_VAR_org_id=
export TF_VAR_service_project_name=
export TF_VAR_host_project_name=${PROVISIONER_PROJECT}
export TF_VAR_region=us-west1
export TF_VAR_billing_account=
export TF_VAR
```

## Terraform Project
```
variable "service_project_name" {}
variable "billing_account" {}
variable "org_id" {}
variable "region" {}

provider "google" {
 region = "${var.region}"
}

#resource "random_id" "id" {
# byte_length = 4
# prefix      = "${var.service_project_name}-"
#}

resource "google_project" "project" {
 name            = "${var.service_project_name}"
 project_id      = "${var.service_project_name}"
 billing_account = "${var.billing_account}"
 org_id          = "${var.org_id}"
}

resource "google_project_services" "project" {
 project = "${google_project.project.project_id}"
 services = [
   "compute.googleapis.com",
   "container.googleapis.com"
 ]
}

# A host project provides network resources to associated service projects.
resource "google_compute_shared_vpc_host_project" "host" {
  project = "var.host_project_name"
}

# A service project gains access to network resources provided by its
# associated host project.
resource "google_compute_shared_vpc_service_project" "service1" {
  host_project    = "${google_compute_shared_vpc_host_project.host.project}"
  service_project = "var.service_project_name"
}


output "project_id" {
 value = "${google_project.project.project_id}"
}
```




