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

az group deployment create `
  --resource-group $RESOURCE_GROUP `
  --template-file .\arm-templates\template.json `
  --handle-extended-json-format `
  --query properties.outputs `
  --parameters .\arm-templates\parameters.json `
  --parameters `
      prefix=$PREFIX
```

The last command shows outputs like below.

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
