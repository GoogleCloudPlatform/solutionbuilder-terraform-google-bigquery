<walkthrough-metadata>
  <meta name="title" content="Edit Jumpstart Solution and deploy tutorial " />
   <meta name="description" content="Make it mine neos tutorial" />
  <meta name="component_id" content="1361081" />
  <meta name="short_id" content="true" />
</walkthrough-metadata>

# Customize a data warehouse With BigQuery Solution

Learn how to build and deploy your own proof of concept solution based on the deployed [Data warehouse with BigQuery](https://console.cloud.google.com/products/solutions/details/data-warehouse) Jump Start Solution and deploy it. You can customize the Jump Start Solution deployment by creating a copy of the source code. You can modify the infrastructure and application code as needed and redeploy the solution with the changes.

To avoid conflicts, only one user should modify and deploy a solution in a single Google Cloud project.

## Open cloned repository as workspace

Open the directory where the repository is cloned as a workspace in the editor, follow the steps based on whether you are using the Cloud Shell Editor in Preview Mode or Legacy Mode.

---
**Legacy Cloud Shell Editor**

1. Go to the `File` menu.
2. Select `Open Workspace`.
3. Choose the directory where the repository has been cloned. This directory is the current directory in the cloud shell terminal.

**New Cloud Shell Editor**

1. Go the hamburger icon located in the top left corner of the editor.
2. Go to the `File` Menu.
3. Select `Open Folder`.
4. Choose the directory where the repository has been cloned. This directory is the current directory in the cloud shell terminal.

## Before you begin

Before editing the solution, you should be aware of the following information:
* Application code is available under `./modules/data_warehouse/src`
* Terraform / infrastructure code is available in the `./modules/data_warehouse/*.tf` files.

We also strongly recommend that you familiarize yourself with the Data warehouse with BigQuery solution by reading the [solution guide](https://cloud.google.com/solutions/data-warehouse).

## Edit the solution

For example, edit the <walkthrough-editor-select-line filePath="./modules/data_warehouse/main.tf" startLine="79" endLine="80" startCharacterOffset="0" endCharacterOffset="0">./modules/data_warehouse/main.tf</walkthrough-editor-select-line> to update labels for resource type: `google_storage_bucket` and name: `raw_bucket` like below

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

Optional: If you want to learn individual steps involved in the script, you can skip this step and continue with the rest of the tutorial. However, if you want an automated deployment without following the full tutorial, run the <walkthrough-editor-open-file filePath="./modules/data_warehouse/deploy_solution.sh">deploy_solution.sh</walkthrough-editor-open-file> script.

```bash
./deploy_solution.sh
```

## Gather information to intialize the deployment environment

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
* The service account has the following format:

```
projects/<var>PROJECT_ID</var>/serviceAccounts/<service-account>@<var>PROJECT_ID</var>.iam.gserviceaccount.com
```

```
Note <service-account> part and set the <var>SERVICE_ACCOUNT</var> value.
You can also set it to any existing service account.
```

---
**Assign the required roles to the service account**
```bash
CURRENT_POLICY=$(gcloud projects get-iam-policy <var>PROJECT_ID</var> --format=json)
MEMBER="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com"
while IFS= read -r role || [[ -n "$role" ]]
do \
if echo "$CURRENT_POLICY" | jq -e --arg role "$role" --arg member "$MEMBER" '.bindings[] | select(.role == $role) | .members[] | select(. == $member)' > /dev/null; then \
    echo "IAM policy binding already exists for member ${MEMBER} and role ${role}"
else \
    gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> \
    --member="$MEMBER" \
    --role="$role" \
    --condition=None
fi
done < "roles.txt"
```

---
**Create a Terraform input file**

Get the existing region being used for terraform resources.

```bash
echo $(gcloud infra-manager deployments describe <var>DEPLOYMENT_NAME</var> --location <var>REGION</var> --format json) | jq -r '.terraformBlueprint.inputValues.region.inputValue'
```

Create an `input.tfvars` file in the current directory with the following contents. Update the region fetched above in the `TF_REGION` variable:

```
region="<var>TF_REGION</var>"
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

**Create the cloud storage bucket if it does not exist already**

Verify if the Cloud Storage bucket exists
```bash
gsutil ls gs://<var>PROJECT_ID</var>_infra_manager_staging
```

If the command returns an error indicating a non-existing bucket, create the bucket by running below command. Otherwise move on to the next step.
```bash
gsutil mb gs://<var>PROJECT_ID</var>_infra_manager_staging/
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

Monitor your deployment at [Solution deployments page](https://console.cloud.google.com/products/solutions/deployments?pageState=(%22deployments%22:(%22f%22:%22%255B%257B_22k_22_3A_22Labels_22_2C_22t_22_3A13_2C_22v_22_3A_22_5C_22modification-reason%2520_3A%2520make-it-mine_5C_22_22_2C_22s_22_3Atrue_2C_22i_22_3A_22deployment.labels_22%257D%255D%22))).

## Save your edits to the solution

Use any of the following methods to save your edits to the solution

---
**Download the solution**

To download your solution, in the `File` menu, select `Download Workspace`. The solution is downloaded in a compressed format.

---
**Save your solution to your Git repository**

Set the remote URL to your Git repository
```bash 
git remote set-url origin [git-repo-url]
```

Review the modified files, commit and push to your remote repository branch.

## Delete the deployed solution

Optional: Use one of the below options in case you want to delete the deployed solution

* Go to [Solution deployments page](https://console.cloud.google.com/products/solutions/deployments?pageState=(%22deployments%22:(%22f%22:%22%255B%257B_22k_22_3A_22Labels_22_2C_22t_22_3A13_2C_22v_22_3A_22_5C_22modification-reason%2520_3A%2520make-it-mine_5C_22_22_2C_22s_22_3Atrue_2C_22i_22_3A_22deployment.labels_22%257D%255D%22))).
* Click on the link under "Deployment name". It will take you to the deployment details page for the solution.
* Click on the "DELETE" button located at the top right corner of the page.

<walkthrough-inline-feedback></walkthrough-inline-feedback>
  
