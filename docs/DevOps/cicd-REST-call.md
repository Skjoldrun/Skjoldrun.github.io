---
layout: page
title: DevOps - CI Pipeline REST call
parent: DevOps
---

# CI Pipeline REST call

Here's how to call a REST Server to retrieve values. These can also be stored in the pipeline variables, even in the secret ones.
Get the server address and endpoint key from the pipeline variables.

```shell
$Server = "$(Server)"
$SecretKey = "$(SecretKey)"

$uri = "https://$($Server)/KeyVault/GetSecretUsingCache?secretKey=$($SecretKey)"
Write-Host "Call from $($uri)"

$response = Invoke-RestMethod `
    -Uri $uri `
    -Method Get `
    -TimeoutSec 30
    
$secretValue = $response.secret

if ([string]::IsNullOrWhiteSpace($secretValue)) {
    throw "Secret was empty or missing in REST response"
}

Write-Host "Secret successfully retrieved"
Write-Host $secretValue # Only for Test usage!!

# Set Azure DevOps pipeline variable (secret)
$pipeSecret = "$(RESTSecret)"
Write-Host "Secret from Pipe Var before: $($pipeSecret.ToCharArray())" # reveals the secret instead of printing ***

Write-Host "##vso[task.setvariable variable=RESTSecret;issecret=true]$secretValue"
Write-Host "Attention: The secret from the pipeline level secured variable can only be used in the next task!"

Write-Host "Done."
```

***Note:* You cannot use the overwritten pipeline var right away after the `##vso[task.setvariable variable=RESTSecret;issecret=true]$secretValue` call. This can only be accessed in the next task!**

```shell
$pipeSecret = "$(RESTSecret)"
Write-Host "Secret from Pipe Var after: $($pipeSecret.ToCharArray())"

Write-Host "Done."
```
