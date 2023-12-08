<walkthrough-metadata>
  <meta name="title" content="Edit Jumpstart Solution and deploy tutorial " />
   <meta name="description" content="Make it mine neos tutorial" />
  <meta name="component_id" content="1361081" />
  <meta name="short_id" content="true" />
</walkthrough-metadata>

# Customize a data warehouse With BigQuery Solution

Learn how to build and deploy your own proof of concept solution based on the deployed [Data warehouse with BigQuery](https://console.cloud.google.com/products/solutions/details/data-warehouse) Jump Start Solution (JSS) and deploy it. You can customize the Jump Start Solution (JSS) deployment by creating a copy of the source code. You can modify the infrastructure and application code as needed and redeploy the solution with the changes.

To avoid conflicts, only one user should modify and deploy a solution in a single GCP project.

NOTE: Open the directory where the repository is cloned as a workspace in the editor:
* Go to the `File` menu.
* Select `Open Workspace`.
* Choose the directory where the repository has been cloned.

## Details of your chosen data warehouse with BigQuery Jump Start Solution 

* [Solution Guide](https://cloud.google.com/solutions/data-warehouse)
* Application code is available under `./modules/data_warehouse/src`
* Terraform / infrastructure code is available in the `./modules/data_warehouse/*.tf` files.

## Edit the solution

Edit the <walkthrough-editor-select-line filePath="./modules/data_warehouse/main.tf" startLine="79" endLine="80" startCharacterOffset="0" endCharacterOffset="0">./modules/data_warehouse/main.tf</walkthrough-editor-select-line> to update labels for resource type: `google_storage_bucket` and name: `raw_bucket` like below

```
 labels = {
    data-warehouse = "true",
    make-it-mine   = "true"
  }
```

NOTE: The changes in infrastructure may lead to reduction or increase in the incurred cost.

---
**Navigate to Data Warehouse module directory**

Use the following command to change the working directory to the `modules/data_warehouse`:
```bash
cd modules/data_warehouse
```

**Create an automated deployment**

(Optional step) If you want to learn individual steps involved in the script, you can skip this step and continue with the rest of the tutorial. However, if you want an automated deployment without following the full tutorial, run the <walkthrough-editor-open-file filePath="./modules/data_warehouse/deploy.sh">deploy.sh</walkthrough-editor-open-file> script.

```bash
./deploy.sh
```

## Gather information to intialize the  gcloud command

---
**Project ID**

Get the Project ID:

```bash
gcloud config get project
```

```
Use the output to set the <var>PROJECT_ID</var>
```
---
**Deployment region**

```
For <var>REGION</var>, provide the region (e.g. us-central1) where you created the deployment resources.
```
---
**Deployment name**

```bash
gcloud infra-manager deployments list --location <var>REGION</var> --filter="labels.goog-solutions-console-deployment-name:* AND labels.goog-solutions-console-solution-id:data-warehouse"
```

```
Use the output value of name to set the <var>DEPLOYMENT_NAME</var>
```

## Deploy the solution


---
**Fetch Deployment details**
```bash
gcloud infra-manager deployments describe <var>DEPLOYMENT_NAME</var> --location <var>REGION</var>
```
From the output, note down the following:
* The values of the existing deployment available in the `terraformBlueprint.inputValues` section.
* The service account. It is of the following form:

```
projects/<var>PROJECT_ID</var>/serviceAccounts/<service-account>@<var>PROJECT_ID</var>.iam.gserviceaccount.com
```

```
Note <service-account> part and set the <var>SERVICE_ACCOUNT</var> value.
You can also set it to any exising service account.
```

---
**Assign the required roles to the service account**
```bash
while IFS= read -r role || [[ -n "$role" ]]
do \
gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> \
  --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" \
  --role="$role"
done < "roles.txt"
```

---
**Create a terraform input file**

Create an `input.tfvars` file in the current directory with the following contents:

```
region="us-central1"
project_id = "<var>PROJECT_ID</var>"
deployment_name = "<var>DEPLOYMENT_NAME</var>"
labels = {
  "goog-solutions-console-deployment-name" = "<var>DEPLOYMENT_NAME</var>",
  "goog-solutions-console-solution-id" = "data-warehouse"
}
force_destroy = true
deletion_protection = false
```

---
**Deploy the solution**

Trigger the re-deployment. 
```bash
gcloud infra-manager deployments apply projects/<var>PROJECT_ID</var>/locations/<var>REGION</var>/deployments/<var>DEPLOYMENT_NAME</var> --service-account projects/<var>PROJECT_ID</var>/serviceAccounts/<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com --local-source="." --inputs-file=./input.tfvars --labels="modification-reason=make-it-mine,goog-solutions-console-deployment-name=<var>DEPLOYMENT_NAME</var>,goog-solutions-console-solution-id=data-warehouse,goog-config-partner=sc"
```

---
**Monitor the deployment**

Get the deployment details.

```bash
gcloud infra-manager deployments describe <var>DEPLOYMENT_NAME</var> --location <var>REGION</var>
```

Monitor your deployment at [JSS deployment page](https://console.cloud.google.com/products/solutions/deployments?pageState=(%22deployments%22:(%22f%22:%22%255B%257B_22k_22_3A_22Labels_22_2C_22t_22_3A13_2C_22v_22_3A_22_5C_22modification-reason%2520_3A%2520make-it-mine_5C_22_22_2C_22s_22_3Atrue_2C_22i_22_3A_22deployment.labels_22%257D%255D%22))).

## Save your edits to the solution

Use any of the following methods to save your edits to the solution

---
**Download the solution**

To download your solution, in the `File` menu, select `Download Workspace`. The solution is downloaded in a compressed format.

---
**Save your solution to your Git repository**

Set the remote url to your Git repository
```bash 
git remote set-url origin [git-repo-url]
```

Review the modified files, commit and push to your remote repository branch.
<walkthrough-inline-feedback></walkthrough-inline-feedback>
  