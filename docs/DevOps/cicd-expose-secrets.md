---
layout: page
title: DevOps - CI/CD Pipeline expose secrets
parent: DevOps
---

# CI/CD Pipeline expose secrets

There is a trick to recover a stored secret variable of the CI/CD pipeline if you can't remember its value. The pipeline tries to hide the value of its secrets in the debugging output, but it can be revealed with the following script:

```shell
Write-Host "This will not be displayed: $($Env:SuperSecret)"
Write-Host "This will not be displayed: $($Env:MappedSecred)"
Write-Host "This will be displayed: $($Env:MappedSecred.ToCharArray())"
```

[![pipeline variables](/assets/images/articles/DevOps/DevOps_expose_secrets.png)](/assets/images/articles/DevOps/DevOps_expose_secrets.png)


[![pipeline variables](/assets/images/articles/DevOps/DevOps_expose_secrets_output.png)](/assets/images/articles/DevOps/DevOps_expose_secrets_output.png)
