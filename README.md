Automated Synthea Generator and FHIR Uploader
---------------------------------------------

This repository builds a Docker image with a Synthea patient generator and a FHIR uploader that can be used with FHIR servers using Azure Active Directory as the OAuth2 provider. Specifically, when started, the Docker container will:

1. Generate the desired number of patients.
2. Authenticate with Azure Active Directory to obtain a token.
3. Upload patients to FHIR server.  

The Azure Active Directory authentication and FHIR server upload is handled by the [`FhirAADUploader`](FhirAADUploader) app, which is a .NET Core command line application.

To generate 100 patients and upload them to a FHIR server with URL `https://my-fhir-server.com`:

```
docker run --name synthea --rm -t \ 
-e AzureAD_ClientSecret='AAD-CLIENT-SECRET' \ 
-e AzureAD_ClientId='AAD-CLIENT-ID' \ 
-e AzureAD_Authority='https://login.microsoftonline.com/TENANT-ID/' \ 
-e AzureAD_Audience='AAD-FHIR-API-APP-ID' -e NUMBER_OF_PATIENTS='100' \ 
-e FHIR_SERVER_URL='https://my-fhir-server/com/' hansenms/synthegenerator
```

To build your own version of the Docker image:

```
docker build -t yourrepo/yourtagname .
```

You can use an [Azure Container Instance](https://azure.microsoft.com/en-us/services/container-instances/) to generate and upload the patients. To run this image using the Azure CLI:

```bash
az container create --resource-group RgName \ 
--image hansenms/syntheagenerator --name ContainerInstanceName \ 
--cpu 2 --memory 4 --restart-policy Never \ 
-e AzureAD_ClientSecret='CLIENT-SECRET' \ 
AzureAD_ClientId='CLIENT-ID' \ 
AzureAD_Authority='https://login.microsoftonline.com/TENANT-ID/' \ 
AzureAD_Audience='FHIR-SERVER-API-APP' \ 
FHIR_SERVER_URL='https://my-fhir-server.com/' \ 
NUMBER_OF_PATIENTS='100'
```

There is also a template included in this repository, which you can deploy with PowerShell or Azure CLI. Edit the `azuredeploy.parameters.json` to match your setup. 

Deploy with PowerShell like this:

```PowerShell
$rg = New-AzureRmResourceGroup -Name RgName -Location eastus

New-AzureRmResourceGroupDeployment -Name synthdeploy ` 
-ResourceGroupName $rg.ResourceGroupName ` 
-TemplateFile .\azuredeploy.json ` 
-TemplateParameterFile .\azuredeploy.parameters.json 
```

Or you can deploy with the Azure CLI:

```bash
#Create a resource group:
RGNAME=MyResourceGroup
az group create --name ${RGNAME} --location eastus

#Deploy using the template
az group deployment create --resource-group ${RGNAME} \ 
--template-file ./azuredeploy.json \ 
--parameters @azuredeploy.parameters.json
```

You can also deploy through the portal. Simply hit the button below and fill in the details:

<a href="https://transmogrify.azurewebsites.net/azuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
