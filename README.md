# Play.Identity

Play Economy Identity microservice.

```powershell
$version="1.0.2"
$owner="icodedotnetmicroservices"
$gh_pat="[PAT HERE]"

dotnet pack src\Play.Identity.Contracts\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/Play.Identity -o ..\packages


```
