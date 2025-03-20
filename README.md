# Bypass limit of path of Front Door / Application Gateway
This sample objective to handle the path limit of 200 origins of Front Door and / or 100 backends of Application Gateway, for very large deployments / migrations that all applications is under a unique URL and distributed by paths

## Architecture diagram

## Prerequisites
- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

---

## Configure networking

Set up the virtual networking for the environment

```bash
az group create \
    --name fd-appg-pahtlimit \
    --location brazilsouth
```