# Play.Identity

Play Economy Identity microservice.

```powershell
$version="1.0.3"
$owner="icodedotnetmicroservices"
$gh_pat="[PAT HERE]"

dotnet pack src\Play.Identity.Contracts\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/Play.Identity -o ..\packages
```

## Build the docker image

```powershell
$env:GH_OWNER="icodedotnetmicroservices"
$env:GH_PAT="[PAT HERE]"

docker build --secret id=GH_OWNER --secret id=GH_PAT -t play.identity:$version .

```

## Run the docker image

```powershell
$adminPass="[PASSWORD HERE]"
$cosmoDbConnString= "[CONN STRING HERE]"
docker run -it --rm -p 5002:5002 --name identity -e MongoDbSettings__ConnectionString=$cosmoDbConnString -e RabbitMqSettings__Host=rabbitmq -e IdentitySettings__AdminUserPassword=$adminPass --network playinfra_default play.identity:$version
```
