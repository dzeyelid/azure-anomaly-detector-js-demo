# Demo of Anomaly Detector from Cognitive Services family

## How to deploy

Run commands below in PowerShell.

```ps1
$RESOURCE_GROUP="<Put a new resource group name>"
$LOCATION="japaneast"  # Put your favorite location
$PREFIX="<Put prefix string>"

az group create `
  --name $RESOURCE_GROUP `
  --location $LOCATION

az group deployment validate `
  --resource-group $RESOURCE_GROUP `
  --template-file .\arm-templates\template.json `
  --handle-extended-json-format `
  --parameters .\arm-templates\parameters.json `
  --parameters `
      prefix=$PREFIX

$deploymentResult=az group deployment create `
  --resource-group $RESOURCE_GROUP `
  --template-file .\arm-templates\template.json `
  --handle-extended-json-format `
  --query properties.outputs `
  --parameters .\arm-templates\parameters.json `
  --parameters `
      prefix=$PREFIX

# Show result
$deploymentResult
```

The `$deploymentResult` shows outputs of the deployment like below.

```json
{
  "anomalyDetectorEndpoint": {
    "type": "String",
    "value": "<Your anomaly detector's endpoint>"
  },
  "anomalyDetectorKey": {
    "type": "String",
    "value": "<Your anomaly detector's primary key>"
  }
}
```

Set environment variables to run a demo of Anomaly Detector locally.

```ps1
$deploymentResult = $deploymentResult | ConvertFrom-Json
$env:VUE_APP_ANOMALY_DETECTOR_KEY=$deploymentResult.anomalyDetectorKey.value
$env:VUE_APP_ANOMALY_DETECTOR_ENDPOINT=$deploymentResult.anomalyDetectorEndpoint.value
```

Or, you can get the primary key and the endpoint like below.

```ps1
$RESOURCE_GROUP="<Put your resource group name>"
$PREFIX="<Put prefix string>"
$ANOMARY_DETECTOR_NAME="${PREFIX}-anomalydetector"

# Get the primary key and set it to an environment variables VUE_APP_ANOMALY_DETECTOR_KEY
$env:VUE_APP_ANOMALY_DETECTOR_KEY=az cognitiveservices account keys list `
  --resource-group $RESOURCE_GROUP `
  --name $ANOMALY_DETECTOR_NAME `
  --query key1 `
  --output tsv

# Get the endpoint and set it to an environment variables VUE_APP_ANOMALY_DETECTOR_ENDPOINT
$env:VUE_APP_ANOMALY_DETECTOR_ENDPOINT=az cognitiveservices account show `
  --resource-group $RESOURCE_GROUP `
  --name $ANOMALY_DETECTOR_NAME `
  --query endpoint `
  --output tsv
```

## Run the app

After setting the environment variables, you can run the app.

You have to register an app to claim oauth0 access token. See below.
- https://docs.microsoft.com/en-us/azure/time-series-insights/time-series-insights-authentication-and-authorization
- https://docs.microsoft.com/en-us/azure/healthcare-apis/get-healthcare-apis-access-token-cli

```ps1
# az ad sp create-for-rbac --name $TSI_AAD_NAME
$TOKEN=az account get-access-token --resource=https://api.timeseries.azure.com/ | ConvertFrom-Json

$env:VUE_APP_TSI_AAD_TOKEN=$TOKEN.accessToken
$env:VUE_APP_TSI_ENV_FQDN=""

cd web/anomaly-detector-js-demo
npm run serve
```

## References

- https://westus2.dev.cognitive.microsoft.com/docs/services/AnomalyDetector/operations/post-timeseries-entire-detect
