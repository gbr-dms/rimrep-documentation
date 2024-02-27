# Rimrep Client Credential Flow

## Client Creation
Clients are created and managed via terraform code. For creating new clients update **keycloak_clients** in `rimrep-terraform/{environment}/keycloak/terraform.auto.tfvars` file of respective environment and apply the changes. Once after applying the changes, **CLIENT_ID** and **CLIENT_SECRET** of the clients will be stored in AWS secrets manager with the name `keycloak-client-credentials-{CLIENT_ID}`. You can use it for M2M flow.

Note: `rimrep` client id is already created in development and staging-v2.
- In development, terraform code for managing client id is in [**keycloak_clients** var](https://github.com/aodn/rimrep-terraform/blob/main/development/keycloak/terraform.auto.tfvars)
- In staging-v2, terraform code for managing client id is [**keycloak_clients** var](https://github.com/aodn/rimrep-terraform/blob/main/staging-new/keycloak/terraform.auto.tfvars)

## JWT Token/Authorization Bearer token Generation

### Pre-requisite
Set the required environment variables.
```
# These are example values from staging-v2
export ENV_NAME=staging-v2
export REALM_NAME=rimrep-staging-v2
export CLIENT_ID=<Get it from AWS secrets manager>
export CLIENT_SECRET=<Get it from AWS secrets manager>
```
Note: Client ids are managed in terraform code. Once clients are created its credentials are added to AWS secrets manager (keycloak-client-credentials-{clientid}) in their respective environments.

|Variable|Development|Staging-v2|Production|
|--------|--------|--------|--------|
| ENV_NAME | development | staging-v2 | |
| REALM_NAME | rimrep-development | rimrep-staging-v2 | |
| CLIENT_ID | Refer keycloak-client-credentials-{clientid} |Refer keycloak-client-credentials-{clientid}||
| CLIENT_SECRET | Refer keycloak-client-credentials-{clientid} |Refer keycloak-client-credentials-{clientid}||


### Curl command to create the token 
```
curl --location --request POST "https://keycloak.$ENV_NAME.reefdata.io/realms/$REALM_NAME/protocol/openid-connect/token" -s \
  --header "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "client_id=$CLIENT_ID" \
  --data-urlencode "client_secret=$CLIENT_SECRET" \
  --data-urlencode "grant_type=client_credentials"
```

### Python code to create token
```
# Python code to create access_token

import requests
import os

client_id = os.environ["CLIENT_ID"]
client_secret = os.environ["CLIENT_SECRET"]
realm_name = os.environ["REALM_NAME"]
environment = os.environ["ENV_NAME"]

url = "https://keycloak.{}.reefdata.io/realms/{}/protocol/openid-connect/token".format(environment, realm_name)
headers = {
    "Content-Type": "application/x-www-form-urlencoded"
}
data = {
    "client_id": client_id,
    "client_secret": client_secret,
    "grant_type": "client_credentials"
}
response = requests.post(url, headers=headers, data=data)

if response.status_code == 200:
    access_token = response.json().get("access_token")
    print("Access Token:", access_token)
else:
    print("Error:", response.text)
```
## How to use the token
The token generated can be used as a regular Authorization bearer. Example curl command below for reference,
```
curl -H "Authorization: Bearer {access_token_from_previous_step}" https://stac.staging-v2.reefdata.io/collections/abs-census
```

## Token rotation
- Login into Keycloak as admin
- Navigate to the realm by choosing the dropdown at left top corner
- Click Clients link in left menu
- Select the Client
- Select the Credentials tab
- Click Regenerate next to Client secret
- Run keycloak terraform workspace to update the secrets manager with new client secret
