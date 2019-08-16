## Grab the variables
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

## create the provisioner service account and download keys
```
gcloud iam service-accounts create provisioner --display-name "provisioner admin account"
gcloud iam service-accounts keys create ${PROVISIONER_CREDS} --iam-account provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com
```

## grant permissions to the service accounts at the account level
```
gcloud projects add-iam-policy-binding ${PROVISIONER_PROJECT} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/viewer
gcloud projects add-iam-policy-binding ${PROVISIONER_PROJECT} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/storage.admin
```

## grant permissions to the service accounts at the org level
```
gcloud organizations add-iam-policy-binding ${PROVISIONER_VAR_org_id} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/resourcemanager.projectCreator
gcloud organizations add-iam-policy-binding ${PROVISIONER_VAR_org_id} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/billing.user
gcloud organizations add-iam-policy-binding ${PROVISIONER_VAR_org_id} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/compute.xpnAdmin

```

## Export credentials
```
export GOOGLE_APPLICATION_CREDENTIALS=${PROVISIONER_CREDS}
export GOOGLE_PROJECT=${PROVISIONER_PROJECT}
```

## Clone the Project Factory Repository
```
git clone https://github.com/terraform-google-modules/terraform-google-project-factory.git .
```

## Create a variables file 
```
cat <<EOF >> terraforms.tfvars
org_id = ""
domain = "example.com"
name = "service-project-001"
project_id = "service-project-001"
shared_vpc = "host-networking-project-01"
billing_account = ""
folder_id = "123156740"
group_name = "gcp-users"
group_role = "roles/owner"
credentials_path = "/home//.config/gcloud/-provisioner-admin.json"
shared_vpc_subnets = ["projects/host-networking-project-01/regions/us-west2/subnetworks/us-west-prd-datacenter-network","projects/host-networking-project-01/regions/us-east1/subnetworks/us-west-prd-datacenter-network"]
EOF







