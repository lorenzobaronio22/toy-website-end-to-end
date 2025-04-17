# Manage end-to-end deployment with Bicep and GitHub Actions

Reference [link](https://learn.microsoft.com/en-us/training/modules/manage-end-end-deployment-scenarios-using-bicep-github-actions)

## Notes

- azure cli login

```shell
az login
```

- create managed identities

```shell
githubOrganizationName='lorenzobaronio22'
githubRepositoryName='toy-website-end-to-end'

# Test Env
testApplicationRegistrationDetails=$(az ad app create --display-name 'toy-website-end-to-end-test')
testApplicationRegistrationObjectId=$(echo $testApplicationRegistrationDetails | jq -r '.id')
testApplicationRegistrationAppId=$(echo $testApplicationRegistrationDetails | jq -r '.appId')

az ad app federated-credential create \
   --id $testApplicationRegistrationObjectId \
   --parameters "{\"name\":\"toy-website-end-to-end-test\",\"issuer\":\"https://token.actions.githubusercontent.com\",\"subject\":\"repo:${githubOrganizationName}/${githubRepositoryName}:environment:Test\",\"audiences\":[\"api://AzureADTokenExchange\"]}"
# {
#   "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#applications('bc5f4337-56b8-46b5-a2f0-daf7569e8c7c')/federatedIdentityCredentials/$entity",
#   "audiences": [
#     "api://AzureADTokenExchange"
#   ],
#   "description": null,
#   "id": "4559e2e3-1b7c-489c-9c23-15aed16f3048",
#   "issuer": "https://token.actions.githubusercontent.com",
#   "name": "toy-website-end-to-end-test",
#   "subject": "repo:lorenzobaronio22/toy-website-end-to-end:environment:Test"
# }

az ad app federated-credential create \
   --id $testApplicationRegistrationObjectId \
   --parameters "{\"name\":\"toy-website-end-to-end-test-branch\",\"issuer\":\"https://token.actions.githubusercontent.com\",\"subject\":\"repo:${githubOrganizationName}/${githubRepositoryName}:ref:refs/heads/main\",\"audiences\":[\"api://AzureADTokenExchange\"]}"
# {
#   "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#applications('bc5f4337-56b8-46b5-a2f0-daf7569e8c7c')/federatedIdentityCredentials/$entity",
#   "audiences": [
#     "api://AzureADTokenExchange"
#   ],
#   "description": null,
#   "id": "9cffdd4a-e72d-4524-a281-372002950167",
#   "issuer": "https://token.actions.githubusercontent.com",
#   "name": "toy-website-end-to-end-test-branch",
#   "subject": "repo:lorenzobaronio22/toy-website-end-to-end:ref:refs/heads/main"
# }

# Prod Env
productionApplicationRegistrationDetails=$(az ad app create --display-name 'toy-website-end-to-end-production')
productionApplicationRegistrationObjectId=$(echo $productionApplicationRegistrationDetails | jq -r '.id')
productionApplicationRegistrationAppId=$(echo $productionApplicationRegistrationDetails | jq -r '.appId')

az ad app federated-credential create \
   --id $productionApplicationRegistrationObjectId \
   --parameters "{\"name\":\"toy-website-end-to-end-production\",\"issuer\":\"https://token.actions.githubusercontent.com\",\"subject\":\"repo:${githubOrganizationName}/${githubRepositoryName}:environment:Production\",\"audiences\":[\"api://AzureADTokenExchange\"]}"
# {
#   "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#applications('6802893c-5425-4d35-807e-671a07025e73')/federatedIdentityCredentials/$entity",
#   "audiences": [
#     "api://AzureADTokenExchange"
#   ],
#   "description": null,
#   "id": "41d41ac8-e056-4721-a116-1e8b3ecc7b05",
#   "issuer": "https://token.actions.githubusercontent.com",
#   "name": "toy-website-end-to-end-production",
#   "subject": "repo:lorenzobaronio22/toy-website-end-to-end:environment:Production"
# }

az ad app federated-credential create \
   --id $productionApplicationRegistrationObjectId \
   --parameters "{\"name\":\"toy-website-end-to-end-production-branch\",\"issuer\":\"https://token.actions.githubusercontent.com\",\"subject\":\"repo:${githubOrganizationName}/${githubRepositoryName}:ref:refs/heads/main\",\"audiences\":[\"api://AzureADTokenExchange\"]}"
# {
#   "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#applications('6802893c-5425-4d35-807e-671a07025e73')/federatedIdentityCredentials/$entity",
#   "audiences": [
#     "api://AzureADTokenExchange"
#   ],
#   "description": null,
#   "id": "783faa9b-4da3-4f7a-96ce-fa7b1bb5919c",
#   "issuer": "https://token.actions.githubusercontent.com",
#   "name": "toy-website-end-to-end-production-branch",
#   "subject": "repo:lorenzobaronio22/toy-website-end-to-end:ref:refs/heads/main"
# }
```

- create resource groups

```shell
# Test Env
testResourceGroupResourceId=$(az group create --name ToyWebsiteTest --location northeurope --query id --output tsv)

az ad sp create --id $testApplicationRegistrationObjectId
az role assignment create \
   --assignee $testApplicationRegistrationAppId \
   --role Contributor \
   --scope $testResourceGroupResourceId

# Prod Env
productionResourceGroupResourceId=$(az group create --name ToyWebsiteProduction --location westus3 --query id --output tsv)

az ad sp create --id $productionApplicationRegistrationObjectId
az role assignment create \
   --assignee $productionApplicationRegistrationAppId \
   --role Contributor \
   --scope $productionResourceGroupResourceId
```

> [!CAUTION]
> `az role assignment create` commands return a `Bad Request` error. So I stopped here and decide to reuse a Service principal created without federeted-credential, just simple secrets credentials.

