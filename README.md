# Play.Identity

Play Economy Identity microservice.

```powershell
$version="1.0.12"
$owner="icodedotnetmicroservices"
$gh_pat="[PAT HERE]"

dotnet pack src\Play.Identity.Contracts\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/Play.Identity -o ..\packages
```

## Build the docker image

```powershell
$env:GH_OWNER="icodedotnetmicroservices"
$env:GH_PAT="[PAT HERE]"
$containerregisteryname = "acrplayeconomy"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$containerregisteryname.azurecr.io/play.identity:$version" .

```

## Run the docker image

```powershell
$adminPass="[PASSWORD HERE]"
$cosmosDbConnString= "[CONN STRING HERE]"
$serviceBusConnString= "[CONN STRING HERE]"
docker run -it --rm -p 5002:5002 --name identity -e MongoDbSettings__ConnectionString=$cosmosDbConnString -e ServiceBusSettings__ConnectionString=$serviceBusConnString -e ServiceSettings__MessageBroker="SERVICEBUS" -e IdentitySettings__AdminUserPassword=$adminPass  play.identity:$version
```

## Publishing The Docker Image

```powershell
az acr login --name $containerregisteryname
docker push "$containerregisteryname.azurecr.io/play.identity:$version"
```

## Create the Kubernetes namespace

```powershell
$namespace="identity"
kubectl create namespace $namespace
```

## Creating The Pod Managed Identity

```powershell
$appname = "playeconomy"
$aksclustername = "aksclusterplayeconomy"
az identity create --resource-group $appname --name $namespace
$IDENTITY_RESOURCE_ID=az identity show -g $appname -n $namespace --query id -otsv

az aks pod-identity add --resource-group $appname --cluster-name $aksclustername --namespace $namespace --name $namespace --identity-resource-id $IDENTITY_RESOURCE_ID
```

## Granting access to Key Vault Secrets

```powershell
$azurekeyvaultname = "azkeyvaultplayeconomy"
$IDENTITY_CLIENT_ID=az identity show -g $appname -n $namespace --query clientId -otsv
az keyvault set-policy -n $azurekeyvaultname --secret-permissions  get list --spn $IDENTITY_CLIENT_ID

```

## Install the Helm Chart

```powershell
$helmUser=[guid]::Empty.Guid
$helmPassword= az acr login --name acr$appname --expose-token --output tsv --query accessToken

$env:HEML_EXPERIMENTAL_OCI=1
helm registry login "acr$appname.azurecr.io" --username $helmUser --password $helmPassword
$chartVersion="0.1.0"
helm upgrade identity-service oci://acr$appname.azurecr.io/helm/microservice --version $chartVersion -f .\helm\values.yaml -n $namespace --install

```
## Required repository secrets for Github workflow
GH_PAT: Created in Github user profile --> Settings -- Developer Settings --> Personal Access Token