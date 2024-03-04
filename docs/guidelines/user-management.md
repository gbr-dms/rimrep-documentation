# User Access Management

This page explains the user access management in Keycloak. There are two kind of users, 
- Federated users from Identity provider
- Local users directly added to Keycloak

## Federated users
It demonstrates how to integrate Keycloak as an Identity Provider (IDP) and managing user access based in Keycloak roles.

### User Management in Keycloak through the AAF Federation
#### Changes in AAF Federation Portal

Create a New client in AAF for the federation as below

![image](https://github.com/aodn/rimrep-dms/assets/126625288/9b64e6d6-bbe7-4a12-b121-6238d1b746ad)

![image](https://github.com/aodn/rimrep-dms/assets/126625288/ee4bc8fb-691f-4aec-bc61-01846e110826)

To register a new OIDC RP please visit:

AAF Production environment 	
https://manager.aaf.edu.au/oidc/clients/new

AAF Test environment	
https://manager.test.aaf.edu.au/oidc/clients/new


Note: Here is the AAF documentation link for new service creation, https://support.aaf.edu.au/support/solutions/articles/19000096640-openid-connect


#### Changes in Keycloak

- Open the Keycloak admin console in your browser.

- Navigate to the "Identity Providers" section under the realm where you want to configure the IDP.

- Click "Add Provider" and select the type of IDP you want to configure (OpenID Connect).


Note: Here the client ID and Client secret need to be created in the AAF Portal.

### Role Assignment in Keycloak
AAF users will be given only default role which does not have access to protected datasets. In order to access protected datasets, respective roles needs to be assigned to users and it can be accomplished through the Terraform code available at https://github.com/aodn/rimrep-terraform/blob/main/development/keycloak/terraform.auto.tfvars.

## Local users

- Local users are created using Terraform modules, please refer [https://github.com/aodn/rimrep-terraform/tree/main/development/keycloak](https://github.com/aodn/rimrep-terraform/blob/main/development/keycloak/terraform.auto.tfvars).
- Ensure the email id is used as username it is to avoid sharing usernames
- Once the users are created, notify them
- When logging in for the first time, users have to use "Forgot Password" option.
- It will send an email to the users and the instructions to reset the password.
- Once after reset, users can use their new password to login


