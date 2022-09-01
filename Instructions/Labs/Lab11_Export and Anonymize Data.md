# Module 5 Lesson 4 Lab 11: Export and Anonymize Data

### Step 1: Review sample anonymization configuration and customize as needed

1.  Microsoft provides a [sample configuration file](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/de-identified-export#configuration-file) to demonstrate how de-identification works between the FHIR service and an ADLS Gen2 account. In real-world use, you would need to review the HIPAA guidelines and customize the configuration file with additional anonymization rules (e.g., rules for redacting/transforming patient names).

More information on HIPAA de-identification rules can be found [here](https://www.hhs.gov/hipaa/for-professionals/privacy/special-topics/de-identification/index.html).

### Step 2: Configure storage account for export

1.  Follow [these instructions](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/de-identified-export) for creating your anonymizationConfig.json file and placing it inside the anonymization container within your "expsa" storage account.
2.  Next, go [here](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/configure-export-data) and follow the instructions for configuring your "expsa" storage account for the \$export operation. Read the note below about the FHIR service managed identity.

Note: When you deployed Azure components in Lab-01, a [managed identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) was automatically enabled for your FHIR service. That managed identity is what needs to be assigned the Storage Blob Data Contributor role in one of the ADLS Gen2 storage accounts in your resource group. The name of this storage account ends with "expsa". Keep in mind that you are configuring the storage account in this step (and not the FHIR service!). In the storage account, be careful not to add a role assignment to a service client or a service principal by mistake!

Review [How to export FHIR data](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/export-data) for more information.

### Step 3: Export anonymized data

1.  Now you will go to the FHIR CALLS collection in Postman and prepare a new \$export request to de-identify and export FHIR data. This API call will cause the FHIR service to create a new folder inside the anonymization container in the "expsa" storage account and export all Resources to the new folder. The PHI in the Resources will be de-identified following the rules defined in the anonymizationConfig.json file.

You can create a new request in Postman by clicking on the existing GET List Patients request and selecting Duplicate from the View more actions menu. Select Rename and name the new request GET Export Anonymized FHIR Data (the GET should already be present on the left).

The general form of the request string will be:

GET {{fhirurl}}/\$export?_container={{containerName}}&_anonymizationConfig={{configFilename}}

It's recommended to add the containerName and configFilename parameters to your fhir-service environment in Postman. Otherwise, you will need to put these names directly in the request string.

1.  In Postman, go to the Authorization tab for the GET Export Anonymized FHIR Data request and make sure that Inherit auth from parent is selected.
2.  Under the Headers tab for the GET Export Anonymized FHIR Data request in Postman, add/modify the three headers as shown below. A screenshot of the Headers tab in Postman is provided for reference.
-   Accept: application/fhir+json
-   Authorization: Bearer {{bearerToken}}
-   Prefer: respond-async

![export-header](media/1f12bd1b726e27f3678a9a871c0953c7.png)

1.  Be sure to Save your GET Export Anonymized FHIR Data request.
2.  Refresh your access token if necessary (use POST AuthorizeGetToken).
3.  Once everything is set up and ready to go, press Send in Postman to initiate the \$export request.

The \$export operation uses the [FHIR Asynchronous Request Pattern](https://hl7.org/fhir/R4/async.html). Detailed information on headers for bulk export operations in FHIR can be found [here](https://hl7.org/Fhir/uv/bulkdata/export/index.html#headers).

1.  You should receive a 202 Accepted response.
2.  Now if you go to your "expsa" storage account, there should be a new folder within the anonymization container. Go to this folder to access the de-identified FHIR data that you just exported (inside the folder, each row will have three dots on the right side - click these three dots and select View/Edit). You will notice that information has been removed/redacted from the FHIR records per the anonymization rules defined in the anonymizationConfig.json file.

### Step 4: Securely transfer the files to the research team

Researchers from outside organizations cannot have direct access to healthcare or payer organizations' Azure tenants. You will need to find a way to securely transfer the anonymized data to these external groups.

1.  Review the [Create SAS Tokens](https://docs.microsoft.com/en-us/azure/cognitive-services/translator/document-translation/create-sas-tokens?tabs=Containers) documentation for setting up a Shared Access Signature (SAS) token. Then, generate a SAS token to allow a research team to access the anonymized data that you exported.

Note: Pay special attention to the "Permissions" section of the above documentation. Reading files and listing files in a container are two different permissions.

What does success look like for Lab-05?

-   Successfully use an anonymization configuration file and the \$export operation to export anonymized data from the FHIR service.
-   Successfully set up a SAS token to allow access to the anonymized data.
