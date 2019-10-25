注意! 完成したソリューションを使用する場合は、TestCLI の`settings.json`のキーで次の項目を入力する必要があります。

```json
{
  "CognitiveServicesKeys": {
    "Vision": "VisionKeyHere"
  },
  "AzureStorage": {
    "ConnectionString": "ConnectionStringHere",
    "BlobContainer": "images"
  },
  "CosmosDB": {
    "EndpointURI": "CosmosURIHere",
    "Key": "CosmosKeyHere",
    "DatabaseName": "images",
    "CollectionName": "metadata"
  }
}
```