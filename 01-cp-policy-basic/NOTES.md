# Basic CP Policy

Goal: Demonsrate how to get Smart-1 Cloud service and work with basic CP policy from Terraform.

## Prerequisites
- Smart-1 Cloud demo tenant
- [this](https://github.com/mkol5222/cp-automation-workshops-2025) Codespace environment

## Smart-1 Cloud demo tenant
- Register new [Infinity Portal](https://portal.checkpoint.com/register) account for your username/e-mail
- Create a new [Smart-1 Cloud](https://portal.checkpoint.com/dashboard/security-management#/welcome) service by choosing the "Start a demo" option
- Navigate to Smart-1 Cloud service [Settings](https://portal.checkpoint.com/dashboard/security-management#/mgmt/) and choose "API & SmartConsole" section
- Open installed SmartConsole and login using connection token copied to clipboard

![alt text](./img/s1c-sc-api.png "SmartConsole token")

Summary: we have create a new Smart-1 Cloud service and connected to it using SmartConsole. We are ready to provide dedicated credentials for automation.

## Automation credentials

Automation account is eaqual to interactive SmartConsole users when it comes to credentials and permissions. It is best practice to create a dedicated user for automation purposes. We will use `Read Write All` permission profile for this user and will authenticate using `API key` method. One has just one credential - API key - instead of suing username and password, but management server will still know the username of user who owns the API key. This is important for auditing purposes and for parallel devops and clickops operations.

- Open SmartConsole and navigate to `Manage > Permissions & Administrators > Administrators`
- This is where you would add a new user, but we will use the existing one due to demo tenant limitation. Select your user and click `Edit`
- In the `General` tab, select `API Key` as authentication method and click `Generate API Key`
- Copy the API key to clipboard and save it in a safe place. You will not be able to see it again.
- In the `Permissions` tab, you would select `Read Write All` permission profile and click `OK`
- API key is active only once we Install database. Choose `Install database..." option in SmartConsole top left corner menu.

![alt text](./img/sc-admin-apikey.png "SmartConsole admin API key")

We have to collect Smart-1 cloud hostname and cloud ID from Infiniti Portal settings section. This is needed to connect to the management server using Terraform and other API clinets. We will also need the API key we just created.

Look at API instructions in the Smart-1 Cloud settings section. 

![alt text](./img/api-use.png)

- in case of URL `https://cp-automation-wo-rj0p0coc.maas.checkpoint.com/a4008bc9-e0b4-4807-ae74-4d4469ff9f7f/web_api/login`:
  - hostname: `cp-automation-wo-rj0p0coc.maas.checkpoint.com`
  - cloud_id: `a4008bc9-e0b4-4807-ae74-4d4469ff9f7f`

We can try our first API call using curl command:
```bash
export CHECKPOINT_SERVER=cp-automation-wo-rj0p0coc.maas.checkpoint.com
export CHECKPOINT_CLOUD_ID=a4008bc9-e0b4-4807-ae74-4d4469ff9f7f
export CHECKPOINT_API_KEY=your_api_key

export PAYLOAD=$(jq -n --arg api_key "${CHECKPOINT_API_KEY}" '{"api-key": $api_key}')

RESPONSE=$(curl -s -X POST "https://${CHECKPOINT_SERVER}/${CHECKPOINT_CLOUD_ID}/web_api/login" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD")
echo $RESPONSE | jq .
```

Assume we want to use obtained token `sid` in the next API calls. We can use `jq` to extract it from the response:
```bash

export SID=$(echo $RESPONSE | jq -r '.sid')

# latest API reference https://sc1.checkpoint.com/documents/latest/APIs/

PAYLOAD=$(jq -n '{"details-level": "full", name: "Network"}')

RESPONSE=$(curl -s -X POST "https://${CHECKPOINT_SERVER}/${CHECKPOINT_CLOUD_ID}/web_api/show-access-rulebase" \
  -H "Content-Type: application/json" \
  -H "X-chkp-sid: ${SID}" \
  -d "$PAYLOAD")
echo $RESPONSE | jq .
```