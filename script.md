# creating the first SPrincipal
az ad sp create-for-rbac --skip-assignment


#creating the resource Group
az group create -n AksTerraform-RG -l eastus2
Location    Name
----------  ---------------
eastus2     AksTerraform-RG
#creating the storage account
az storage account create -n <YOUR_STORAGE_ACCOUNT_NAME> -g AksTerraform-RG -l eastus2
#creating a tfstate container
az storage container create -n tfstate --account-name <YOUR_STORAGE_ACCOUNT_NAME>
#creating the KeyVault
az keyvault create -n <YOUR_KV_NAME> -g AksTerraform-RG -l eastus2
Location    Name            ResourceGroup
----------  --------------  ---------------
eastus2     <YOUR_KV_NAME>  AksTerraform-RG
#Creating a SAS Token for the storage account, storing in KeyVault
az storage container generate-sas --account-name <YOUR_STORAGE_ACCOUNT_NAME> --expiry 2020-01-01 --name tfstate --permissions dlrw -o json | xargs az keyvault secret set --vault-name <YOUR_KV_NAME> --name TerraformSASToken --value
#creating a Service Principal for AKS and Azure DevOps
az ad sp create-for-rbac -n "AksTerraformSPN"
#creating an ssh key if you don't already have one
ssh-keygen  -f ~/.ssh/id_rsa_terraform
#store the public key in Azure KeyVault
az keyvault secret set --vault-name <YOUR_KV_NAME> --name LinuxSSHPubKey -f ~/.ssh/id_rsa_terraform.pub > /dev/null
#store the service principal id in Azure KeyVault
az keyvault secret set --vault-name <YOUR_KV_NAME> --name spn-id --value <SPN_ID> /dev/null
#store the service principal secret in Azure KeyVault
az keyvault secret set --vault-name <YOUR_KV_NAME> --name spn-secret --value <SPN_SECRET> /dev/null