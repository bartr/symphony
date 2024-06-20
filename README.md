# Symphony PoC

## Links

- Getting Started: <https://aepreviews.ms/docs/toolchain-orchestrator/getting-started/>
- Scenarios: <https://aepreviews.ms/docs/toolchain-orchestrator/sample-scenarios/scenario-1-deploying-prometheus-to-a-k8s-cluster/>

```powershell

az connectedk8s list

az resource list -g $resourceGroup --resource-type "$providerName/solutions" -o table

az resource show -g $resourceGroup --resource-type "$providerName/solutions" -n heartbeat

az resource list -g $resourceGroup --resource-type "$providerName/solutions/versions" -o table

az resource show -g $resourceGroup --namespace $providerName --resource-type versions -n v101 --parent "solutions/heartbeat"

```

```powershell

# edit these as necessary
$subId = "af54d2ce-0dcb-48f8-9d2d-ff9c53e48c8d"
$resourceGroup = "bugbashBart"

$arcCluster = "control"
$customLocationName = "control"
$customlocation = "/subscriptions/$subId/resourceGroups/$resourcegroup/providers/Microsoft.ExtendedLocation/customLocations/$customLocationName"

$extensionName = "symphonyext"
$extensionVersion = "0.0.9"
$location = "eastus2"
$targetVersionName = "v1"
$providerName="Microsoft.ToolchainOrchestrator"

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


# use the control cluster
kuse control

# create heartbeat solution / version

$arcCluster = "control"
$customLocationName = "control"
$customlocation = "/subscriptions/$subId/resourceGroups/$resourcegroup/providers/Microsoft.ExtendedLocation/customLocations/$customLocationName"

$solutionName = "heartbeat"

$solutionBody = '{
    "properties": {
    },
    "extendedLocation": {
        "name": "' + $customlocation + '",
        "type": "CustomLocation"
    },
    "tags": {
        "type": "add-in",
        "namespace": "heartbeat",
        "owner": "platform"
    },
    "location": "' + $location + '"
}'
Set-Content -Value $solutionBody -Path ".\$solutionName.json"
az resource create --resource-group $resourceGroup --resource-type "$providerName/solutions" -n $solutionName --is-full-object --properties "@$solutionName.json" --verbose


$solutionVersionName = "v100"
$solutionVersionBody = '{
  "properties": {
    "metadata" : {},
    "components" : []
  },
  "extendedLocation": {
    "name": "' + $customlocation + '",
    "type": "CustomLocation"
  },
  "tags": {
    "version": "1.0.0",
    "container": "ghcr.io/cse-labs/heartbeat:1.0.0",
    "expression": "/c/*"
  },
  "location": "' + $location + '"
}'
Set-Content -Value $solutionVersionBody -Path ".\$solutionName-$solutionVersionName.json"

az resource create --resource-group $resourceGroup --namespace "$providerName" -n $solutionVersionName --resource-type versions --parent "solutions/$solutionName" --is-full-object --properties "@$solutionName-$solutionVersionName.json" --location $location --verbose



$solutionVersionName = "v101"
$solutionVersionBody = '{
  "properties": {
    "metadata" : {},
    "components" : []
  },
  "extendedLocation": {
    "name": "' + $customlocation + '",
    "type": "CustomLocation"
  },
  "tags": {
    "version": "1.0.1",
    "container": "ghcr.io/cse-labs/heartbeat:1.0.1",
    "expression": "/m/type/lab"
  },
  "location": "' + $location + '"
}'
Set-Content -Value $solutionVersionBody -Path ".\$solutionName-$solutionVersionName.json"

az resource create --resource-group $resourceGroup --namespace "$providerName" -n $solutionVersionName --resource-type versions --parent "solutions/$solutionName" --is-full-object --properties "@$solutionName-$solutionVersionName.json" --location $location --verbose


# create lab01 target
$targetName = "lab01"
$customLocationName = $targetName
$customlocation = "/subscriptions/$subId/resourceGroups/$resourcegroup/providers/Microsoft.ExtendedLocation/customLocations/$customLocationName"

$targetBody = '{
    "properties": {
    },
    "name": "' + $targetName + '",
    "extendedLocation": {
    "name": "' + $customlocation + '",
    "type": "CustomLocation"
    },
    "tags": {},
    "location": "' + $location + '"
}'
Set-Content -Value $targetBody -Path ".\$targetName.json"
az resource create --resource-group $resourceGroup --resource-type "$providerName/targets" -n $targetName --is-full-object --properties "@$targetName.json" --verbose


$targetVersionName = "v1"
$targetVersionBody = '{
    "properties": {
        "topologies" : [
            {
                "bindings" : [
                    {
                        "role" : "k8s.container",
                        "provider" : "providers.target.k8s",
                        "config" : {
                        "inCluster" : "true"
                        }
                    }
                ]
            }
        ]
    },
    "extendedLocation": {
        "name": "' + $customlocation + '",
        "type": "CustomLocation"
    },
    "tags": {},
    "location": "' + $location + '"
}'

Set-Content -Value $targetVersionBody -Path ".\$targetName-$targetVersionName.json"

az resource create --resource-group $resourceGroup --namespace "$providerName" -n $targetVersionName --resource-type versions --parent "targets/$targetName" --is-full-object --properties "@$targetName-$targetVersionName.json" --location $location --verbose

az resource show --resource-group $resourceGroup --namespace "$providerName" -n $targetVersionName --resource-type versions --parent "targets/$targetName" --verbose



# create lab02 target
$targetName = "lab02"
$customLocationName = $targetName
$customlocation = "/subscriptions/$subId/resourceGroups/$resourcegroup/providers/Microsoft.ExtendedLocation/customLocations/$customLocationName"

$targetBody = '{
    "properties": {
    },
    "name": "' + $targetName + '",
    "extendedLocation": {
    "name": "' + $customlocation + '",
    "type": "CustomLocation"
    },
    "tags": {},
    "location": "' + $location + '"
}'
Set-Content -Value $targetBody -Path ".\$targetName.json"
az resource create --resource-group $resourceGroup --resource-type "$providerName/targets" -n $targetName --is-full-object --properties "@$targetName.json" --verbose


$targetVersionName = "v1"
$targetVersionBody = '{
    "properties": {
        "topologies" : [
            {
                "bindings" : [
                    {
                        "role" : "k8s.container",
                        "provider" : "providers.target.k8s",
                        "config" : {
                        "inCluster" : "true"
                        }
                    }
                ]
            }
        ]
    },
    "extendedLocation": {
        "name": "' + $customlocation + '",
        "type": "CustomLocation"
    },
    "tags": {},
    "location": "' + $location + '"
}'

Set-Content -Value $targetVersionBody -Path ".\$targetName-$targetVersionName.json"

az resource create --resource-group $resourceGroup --namespace "$providerName" -n $targetVersionName --resource-type versions --parent "targets/$targetName" --is-full-object --properties "@$targetName-$targetVersionName.json" --location $location --verbose

az resource show --resource-group $resourceGroup --namespace "$providerName" -n $targetVersionName --resource-type versions --parent "targets/$targetName" --verbose







$instanceScopeName = "heartbeat"
$instanceVersionName = "v1"

az resource create --resource-group $resourceGroup --resource-type "$providerName/solutions" -n $solutionName --is-full-object --properties "@$solutionName.json" --verbose



$targetName = "lab01"
$customLocationName = $targetName
$customlocation = "/subscriptions/$subId/resourceGroups/$resourcegroup/providers/Microsoft.ExtendedLocation/customLocations/$customLocationName"
$instanceName = $targetName + "instance"
$instanceBody = '{
    "properties": {
    },
    "extendedLocation": {
        "name": "' + $customlocation + '",
        "type": "CustomLocation"
    },
    "tags": {},
    "location": "' + $location + '"
}'
Set-Content -Value $instanceBody -Path ".\$instanceName.json"
az resource create --resource-group $resourceGroup --resource-type "$providerName/instances" -n $instanceName --is-full-object --properties "@$instanceName.json" --verbose


$instanceScopeName = "heartbeat"
$instanceVersionName = "v1"
$instanceVersionBody = '{
  "properties": {
    "scope": "' + $instanceScopeName + '",
    "solution": "' + $solutionName + ':' + $solutionVersionName + '",
    "target": {
        "name": "' + $targetName + ':' + $targetVersionName + '"
    }
  },
  "extendedLocation": {
    "name": "' + $customlocation + '",
    "type": "CustomLocation"
  },
  "tags": {},
  "location": "' + $location + '"
}'
Set-Content -Value $instanceVersionBody -Path ".\$instanceName-$instanceVersionName.json"

az resource create --resource-group $resourceGroup --namespace "$providerName" -n $instanceVersionName --resource-type versions --parent "instances/$instanceName" --is-full-object --properties "@$instanceName-$instanceVersionName.json" --location $location --verbose

az resource show --resource-group $resourceGroup --namespace "$providerName" -n $instanceVersionName --resource-type versions --parent "instances/$instanceName" --verbose

kubectl get deployment -n $instanceScopeName

```
