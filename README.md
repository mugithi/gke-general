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

## grant permissions to the service accounts at the project level
```
gcloud projects add-iam-policy-binding ${PROVISIONER_PROJECT} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/storage.admin
gcloud projects add-iam-policy-binding ${PROVISIONER_PROJECT} --member serviceAccount:provisioner@${PROVISIONER_PROJECT}.iam.gserviceaccount.com --role roles/resourcemanager.projectIamAdmin
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
```



```
diff -r terraform-google-kubernetes-engine/cluster_regional.tf terraform-google-kubernetes-engine.2/cluster_regional.tf
147a148,149
>   max_pods_per_node = lookup(var.node_pools[count.index], "max_pods_per_node", 110)
> 
diff -r terraform-google-kubernetes-engine/examples/shared_vpc_private/main.tf terraform-google-kubernetes-engine.2/examples/shared_vpc_private/main.tf
60a61
>   max_pods_per_node       = 16
diff -r terraform-google-kubernetes-engine/modules/beta-private-cluster/cluster_regional.tf terraform-google-kubernetes-engine.2/modules/beta-private-cluster/cluster_regional.tf
191a192,193
>   max_pods_per_node = lookup(var.node_pools[count.index], "max_pods_per_node", 110)
> 
diff -r terraform-google-kubernetes-engine/modules/beta-private-cluster/cluster_zonal.tf terraform-google-kubernetes-engine.2/modules/beta-private-cluster/cluster_zonal.tf
188c188
<   max_pods_per_node = lookup(var.node_pools[count.index], "max_pods_per_node", 250)
---
>   max_pods_per_node = lookup(var.node_pools[count.index], "max_pods_per_node", 110)
diff -r terraform-google-kubernetes-engine/modules/beta-private-cluster/main.tf terraform-google-kubernetes-engine.2/modules/beta-private-cluster/main.tf
227,238d226
<   # BETA features
<   cluster_type_output_istio_enabled = {
<     regional = element(concat(google_container_cluster.primary.*.addons_config.0.istio_config.0.disabled, [""]), 0)
<     zonal    = element(concat(google_container_cluster.zonal_primary.*.addons_config.0.istio_config.0.disabled, [""]), 0)
<   }
< 
<   cluster_type_output_pod_security_policy_enabled = {
<     regional = element(concat(google_container_cluster.primary.*.pod_security_policy_config.0.enabled, [""]), 0)
<     zonal    = element(concat(google_container_cluster.zonal_primary.*.pod_security_policy_config.0.enabled, [""]), 0)
<   }
<   # /BETA features
< 
269,273d256
<   # BETA features
<   cluster_istio_enabled               = ! local.cluster_type_output_istio_enabled[local.cluster_type]
<   cluster_cloudrun_enabled            = var.cloudrun
<   cluster_pod_security_policy_enabled = local.cluster_type_output_pod_security_policy_enabled[local.cluster_type]
<   # /BETA features
diff -r terraform-google-kubernetes-engine/modules/beta-private-cluster/outputs.tf terraform-google-kubernetes-engine.2/modules/beta-private-cluster/outputs.tf
116,129d115
< output "istio_enabled" {
<   description = "Whether Istio is enabled"
<   value       = local.cluster_istio_enabled
< }
< 
< output "cloudrun_enabled" {
<   description = "Whether CloudRun enabled"
<   value       = local.cluster_cloudrun_enabled
< }
< 
< output "pod_security_policy_enabled" {
<   description = "Whether pod security policy is enabled"
<   value       = local.cluster_pod_security_policy_enabled
< }
```





