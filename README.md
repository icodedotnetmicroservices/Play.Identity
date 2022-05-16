# Play.Identity

Play Economy Identity microservice.

```powershell
$version="1.0.6"
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
# $containerregisteryname = "acrplayeconomy"
az acr login --name $containerregisteryname
# docker tag play.identity:$version "$containerregisteryname.azurecr.io/play.identity:$version"
docker push "$containerregisteryname.azurecr.io/play.identity:$version"
```

## Create the Kubernetes namespace

```powershell
$namespace="identity"
kubectl create namespace $namespace
```

## Create the Kubernetes secrets

```powershell
kubectl create secret generic identity-secrets --from-literal=cosmosdb-connectionstring=$cosmosDbConnString --from-literal=servicebus-connectionstring=$serviceBusConnString --from-literal=admin-password=$adminPass -n $namespace
```

## Create the Kubernetes pod

```powershell
kubectl apply -f .\kubernetes\identity.yaml -n $namespace
```
