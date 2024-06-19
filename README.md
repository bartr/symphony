# Symphony PoC

## Links

- Getting Started: <https://aepreviews.ms/docs/toolchain-orchestrator/getting-started/>
- Scenarios: <https://aepreviews.ms/docs/toolchain-orchestrator/sample-scenarios/scenario-1-deploying-prometheus-to-a-k8s-cluster/>

```powershell

$arcCluster = "control"
$customLocationName = "control"

$resourceGroup = "bugbashBart"
$subId = "af54d2ce-0dcb-48f8-9d2d-ff9c53e48c8d"
$extensionName = "symphonyext"
$extensionVersion = "0.0.9"
$location = "eastus2"
$targetVersionName = "v1"

$providerName="Microsoft.ToolchainOrchestrator"
$customlocation = "/subscriptions/$subId/resourceGroups/$resourcegroup/providers/Microsoft.ExtendedLocation/customLocations/$customLocationName"


# create cluster
az aks create --name $arcCluster --resource-group $resourceGroup --os-sku AzureLinux --generate-ssh-key

# get kube config
az aks get-credentials --name $arcCluster --resource-group $resourceGroup --overwrite-existing

# connect cluster
az connectedk8s connect -g $resourceGroup -n $arcCluster --location $location

# install toolchain orchestrator
az k8s-extension create --resource-group $resourceGroup --cluster-name $arcCluster --cluster-type connectedClusters --name $extensionName --extension-type Microsoft.ToolchainOrchestrator --scope cluster --release-train dev --version $extensionVersion --auto-upgrade false

# create custom location
az connectedk8s enable-features -n $arcCluster -g $resourceGroup --features cluster-connect custom-locations
az customlocation create -n $customLocationName -g  $resourceGroup --namespace $customLocationName --host-resource-id  "/subscriptions/$subId/resourceGroups/$resourceGroup/providers/Microsoft.Kubernetes/connectedClusters/$arcCluster" --cluster-extension-ids "/subscriptions/$subId/resourceGroups/$resourceGroup/providers/Microsoft.Kubernetes/connectedClusters/$arcCluster/Providers/Microsoft.KubernetesConfiguration/extensions/$extensionName" --location $location


$arcCluster = "lab01"
$customLocationName = "lab01"
$customlocation = "/subscriptions/$subId/resourceGroups/$resourcegroup/providers/Microsoft.ExtendedLocation/customLocations/$customLocationName"

# setup cluster


$arcCluster = "lab02"
$customLocationName = "lab02"
$customlocation = "/subscriptions/$subId/resourceGroups/$resourcegroup/providers/Microsoft.ExtendedLocation/customLocations/$customLocationName"

# setup cluster


kuse control

$targetName = "lab01"
$solutionName = "heartbeat"
$solutionVersionName = "v1"

# create lab01 target / version

# create heartbeat solution / version

# create instance / version


az resource create --resource-group $resourceGroup --namespace "$providerName" -n $solutionName$solutionVersionName --resource-type versions --parent "solutions/$solutionName" --is-full-object --properties "@$solutionName-$solutionVersionName.json" --location $location --verbose


$instanceName = "lab01"
az resource create --resource-group $resourceGroup --resource-type "$providerName/instances" -n $instanceName --is-full-object --properties "@$instanceName-instance.json" --verbose


$instanceScopeName = "heartbeat"
$instanceVersionName = "v1"
az resource create --resource-group $resourceGroup --namespace "$providerName" -n $instanceVersionName --resource-type versions --parent "instances/$instanceName" --is-full-object --properties "@$instanceName-instance-$instanceVersionName.json" --location $location --verbose


az resource show --resource-group $resourceGroup --namespace "$providerName" -n $instanceVersionName --resource-type versions --parent "instances/$instanceName" --verbose




$targetName = "control"
$solutionName = "heartbeat"
$instanceName = "controlinstance"
$instanceScopeName = "heartbeat"
$instanceVersionName = "v1"

az resource create --resource-group $resourceGroup --resource-type "$providerName/solutions" -n $solutionName --is-full-object --properties "@$solutionName.json" --verbose



$instanceName = "lab01instance"

```
