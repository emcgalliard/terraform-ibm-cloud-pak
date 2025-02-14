# Terraform Module to install Cloud Pak for Data

This Terraform Module installs **Cloud Pak for Data** on an Openshift (ROKS) cluster on IBM Cloud.

**Module Source**: `github.com/ibm-build-lab/terraform-ibm-cloud-pak.git//modules/cp4data_3.5`

- [Terraform Module to install Cloud Pak for Data](#terraform-module-to-install-cloud-pak-for-data)
  - [Set up access to IBM Cloud](#set-up-access-to-ibm-cloud)
  - [Provisioning this module in a Terraform Script](#provisioning-this-module-in-a-terraform-script)
    - [Setting up the OpenShift cluster](#setting-up-the-openshift-cluster)
    - [Installing the CP4Data Module](#installing-the-cp4data-module)
  - [Input Variables](#input-variables)
  - [Executing the Terraform Script](#executing-the-terraform-script)
  - [Accessing the Cloud Pak Console](#accessing-the-cloud-pak-console)
  - [Clean up](#clean-up)
  - [Troubleshooting](#troubleshooting)
  
## Set up access to IBM Cloud

If running these modules from your local terminal, you need to set the credentials to access IBM Cloud.

Go [here](../CREDENTIALS.md) for details.

## Provisioning this module in a Terraform Script

In your Terraform code define the `ibm` provisioner block with the `region` and the `generation`, which is **1 for Classic** and **2 for VPC Gen 2**.

```hcl
provider "ibm" {
  region     = "us-south"
}
```

### Setting up the OpenShift cluster

NOTE: an OpenShift cluster is required to install the Cloud Pak. This can be an existing cluster or can be provisioned using our `roks` Terraform module.

To provision a new cluster, refer [here](https://github.com/ibm-build-lab/terraform-ibm-cloud-pak/tree/main/modules/roks#building-a-new-roks-cluster) for the code to add to your Terraform script. The recommended size for an OpenShift 4.5+ cluster on IBM Cloud Classic contains `4` workers of flavor `b3c.16x64`, however read the [Cloud Pak for Data documentation](https://www.ibm.com/docs/en/cloud-paks/cp-data) to confirm these parameters or if you are using IBM Cloud VPC or a different OpenShift version.

Add the following code to get the OpenShift cluster (new or existing) configuration:

```hcl
data "ibm_resource_group" "group" {
  name = var.resource_group
}

data "ibm_container_cluster_config" "cluster_config" {
  cluster_name_id   = var.cluster_name_id
  resource_group_id = data.ibm_resource_group.group.id
  download          = true
  config_dir        = "./kube/config"     // Create this directory in advance
  admin             = false
  network           = false
}
```

**NOTE**: Create the `./kube/config` directory if it doesn't exist.

Input:

- `cluster_name_id`: either the cluster name or ID.

- `ibm_resource_group`:  resource group where the cluster is running

Output:

`ibm_container_cluster_config` used as input for the `cp4data` module

### Installing the CP4Data Module

Use a `module` block assigning `source` to `github.com/ibm-build-lab/terraform-ibm-cloud-pak.git//cp4data_3.5`. Then set the [input variables](#input-variables) required to install the Cloud Pak for Data.

```hcl
module "cp4data" {
  source          = "./.."
  enable          = var.enable

  // ROKS cluster parameters:
  openshift_version   = var.openshift_version
  cluster_config_path = data.ibm_container_cluster_config.cluster_config.config_file_path
  on_vpc              = var.on_vpc
  portworx_is_ready   = var.portworx_is_ready // only need if on_vpc = true
  
  // Prereqs
  worker_node_flavor = var.worker_node_flavor

  // Entitled Registry parameters:
  entitled_registry_key        = var.entitled_registry_key
  entitled_registry_user_email = var.entitled_registry_user_email

  // CP4D License Acceptance
  accept_cpd_license = var.accept_cpd_license

  // CP4D Info
  cpd_project_name = var.cpd_project_name

  // Parameters to install submodules
  install_watson_knowledge_catalog = var.install_watson_knowledge_catalog
  install_watson_studio            = var.install_watson_studio
  install_watson_machine_learning  = var.install_watson_machine_learning
  install_watson_open_scale        = var.install_watson_open_scale
  install_data_virtualization      = var.install_data_virtualization
  install_streams                  = var.install_streams
  install_analytics_dashboard      = var.install_analytics_dashboard
  install_spark                    = var.install_spark
  install_db2_warehouse            = var.install_db2_warehouse
  install_db2_data_gate            = var.install_db2_data_gate
  install_big_sql                  = var.install_big_sql
  install_rstudio                  = var.install_rstudio
  install_db2_data_management      = var.install_db2_data_management
}
```



- 

## Input Variables

| Name                               | Description                                                                                                                                                                                                                | Default                     | Required |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | -------- |
| `enable`                           | If set to `false` does not install the cloud pak on the given cluster. By default it's enabled                                                                                                                        | `true`                      | No       |
| `on_vpc`                           | If set to `false`, it will set the install do classic ROKS. By default it's disabled                                                                                                                        | `false`                      | No       |
| `openshift_version`                | Openshift version installed in the cluster                                                                                                                                                                                 | `4.6`                       | No       |
| `entitled_registry_key`            | Get the entitlement key from https://myibm.ibm.com/products-services/containerlibrary and assign it to this variable. Optionally you can store the key in a file and use the `file()` function to get the file content/key |                             | Yes      |
| `entitled_registry_user_email`     | IBM Container Registry (ICR) username which is the email address of the owner of the Entitled Registry Key                                                                                                                 |                             | Yes      |
| `worker_node_flavor`          | Flavor used to determine worker node hardware for the cluster |  | Yes       |
| `accept_cpd_license`          | If set to `true`, you accept all cpd license agreements including additional modules installed. By default, it's `false` | `false` | Yes       |
| `install_watson_knowledge_catalog` | Install Watson Knowledge Catalog module. By default it's not installed.                                                                                                                                                    | `false`                     | No       |
| `install_watson_studio`            | Install Watson Studio module. By default it's not installed.                                                                                                                                                               | `false`                     | No       |
| `install_watson_machine_learning`  | Install Watson Machine Learning module. By default it's not installed.                                                                                                                                                     | `false`                     | No       |
| `install_watson_open_scale`        | Install Watson Open Scale module. By default it's not installed.                                                                                                                                                           | `false`                     | No       |
| `install_data_virtualization`      | Install Data Virtualization module. By default it's not installed.                                                                                                                                                         | `false`                     | No       |
| `install_streams`                  | Install Streams module. By default it's not installed.                                                                                                                                                                     | `false`                     | No       |
| `install_analytics_dashboard`      | Install Analytics Dashboard module. By default it's not installed.                                                                                                                                                         | `false`                     | No       |
| `install_spark`                    | Install Analytics Engine powered by Apache Spark module. By default it's not installed.                                                                                                                                    | `false`                     | No       |
| `install_db2_warehouse`            | Install DB2 Warehouse module. By default it's not installed.                                                                                                                                                               | `false`                     | No       |
| `install_db2_data_gate`            | Install DB2 Data_Gate module. By default it's not installed.                                                                                                                                                               | `false`                     | No       |
| `install_big_sql`                  | Install Big SQL module. By default it's not installed.                                                                                                                                                                     | `false`                     | No       |
| `install_rstudio`                  | Install RStudio module. By default it's not installed.                                                                                                                                                                     | `false`                     | No       |
| `install_db2_data_management`      | Install DB2 Data Management module. By default it's not installed.                                                                                                                                                         | `false`                     | No       |

**NOTE** The boolean input variable `enable` is used to enable/disable the module. This parameter may be deprecated when Terraform 0.12 is not longer supported. In Terraform 0.13, the block parameter `count` can be used to define how many instances of the module are needed. If set to zero the module won't be created.

For an example of how to put all this together, refer to our [Cloud Pak for Data Terraform script](https://github.com/ibm-build-lab/cloud-pak-sandboxes/tree/master/terraform/cp4data).

## Executing the Terraform Script

Execute the following commands to install the Cloud Pak:

```bash
terraform init
terraform plan
terraform apply
```

## Accessing the Cloud Pak Console

After execution has completed, access the cluster using `kubectl` or `oc`:

```bash
ibmcloud oc cluster config -c <cluster-name> --admin
kubectl get route -n ibm-common-services cp-console -o jsonpath=‘{.spec.host}’ && echo
```

To get default login id:

```bash
kubectl -n ibm-common-services get secret platform-auth-idp-credentials -o jsonpath='{.data.admin_username}\' | base64 -d && echo
```

To get default Password:

```bash
kubectl -n ibm-common-services get secret platform-auth-idp-credentials -o jsonpath='{.data.admin_password}' | base64 -d && echo
```

## Clean up

When you finish using the cluster, release the resources by executing the following command:

```bash
terraform destroy
```

## Troubleshooting

- Once `module.cpd_install.null_resource.install_cpd` completes. You can check the logs to find out more information about the installation of Cloud Pak for Data.

```bash
cpd-meta-operator: oc -n cpd-meta-ops logs -f deploy/ibm-cp-data-operator

cpd-install-operator: oc -n cpd-tenant logs -f deploy/cpd-install-operator
```
