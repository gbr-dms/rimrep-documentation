# Admin CLI Access Instructions

To access the Admin CLI for Keycloak, follow the steps outlined below:

## Step 1: Obtain Admin CLI Access Token

Run the following command in your terminal to obtain the access token needed for authentication:


access_token=$(curl --location 'https://keycloak.reefdata.io/realms/master/protocol/openid-connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=admin-cli' \
--data-urlencode 'client_secret=<ADMIN_CLI_CLIENT_SECRET>' \
--data-urlencode 'username=admin' \
--data-urlencode 'password=<ADMIN_PASSWORD>' \
--data-urlencode 'grant_type=password' | jq -r '.access_token')

---

Replace <ADMIN_CLI_CLIENT_SECRET> with the client secret of your Admin CLI and <ADMIN_PASSWORD> with your admin password.


### Additional Notes:
Admin Credentials: The admin credentials required for Step 1 can be fetched from the production AWS Secret Manager.
Token Expiry: The access token has a default expiry of 10 minutes. If necessary, you can adjust this setting in the Keycloak admin realm settings.
Please ensure proper authorization and security measures are in place when accessing sensitive data and APIs.

![image](./images/keycloak-admin-api-access-additional-note.png)

---
### Step 2: Access Admin API

Once you have obtained the access token, you can use it to make requests to the Admin API. Use the following command to retrieve information about users, for example:


curl --location 'https://keycloak.reefdata.io/admin/realms/rimrep-production/users' \
--header "Authorization: Bearer $access_token"

Remember to replace rimrep-production with the appropriate realm if needed.


