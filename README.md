# Demo of Anomaly Detector from Cognitive Services family

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

Set environment variables to run demo of Anomaly Detector locally.

```ps1
$deploymentResult = $deploymentResult | ConvertFrom-Json
$env:VUE_APP_ANOMALY_DETECTOR_KEY=$deploymentResult.anomalyDetectorKey.value
$env:VUE_APP_ANOMALY_DETECTOR_ENDPOINT=$deploymentResult.anomalyDetectorEndpoint.value
```
