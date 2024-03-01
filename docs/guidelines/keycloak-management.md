# Keycloak Management
Keycloak management tasks are listed below,
- Adding admin email id to master realm
- Background image update for login page

## Admin User Email Update
Keycloak is configured as IAC though but adding email id for admin user needs to be done manually are listed below,
- Login into Keycloak as admin
- You will be in master realm. If not choose master realm
- Click Users from left hand side menu
- Click admin from the user list
- Update the Email id field of admin
- Click Email verified
- Click Save to save the changes

## Background image update
Code for backgroun image in Keycloak login page is managed in rimrep-flux repo. Details related to background image update are captured here. Below details will help to understand how it is set-up and also to debug issues if custom background image (one with clown fish) is broken or not loading.
- Custom background image (one with clown fish) is loaded to Keycloak via themes
- A new theme called custom theme is added to keycloak and will be loaded to Keycloak container via config map
- Two files are needed for adding the background image for the custom theme, `theme.properties` and `login.css`
-  `theme.properties` is a small file with few lines explains which parent theme is used, where the base theme is imported from and the css file
-  `login.css` is the stylesheet for the custom theme. It is copied from base Keycloak theme by viewing the page source when loading keycloak login page using keycloak theme.
-  Background image and header font styles are updated in `login.css`. Lines updated are marked with a comment `/* Updated */` and `/* Added */` for future reference.
-  Once after added the theme, realm settings to be updated to use the new custom theme and can be done using `rimrep-terraform`
- PR reference,
  - https://github.com/aodn/rimrep-flux/pull/334
  - https://github.com/aodn/rimrep-flux/pull/337
  - https://github.com/aodn/rimrep-terraform/pull/287

**Important Note:** 
During Keycloak upgrade, if background image failed to load it could be because of `login.css` from base theme is updated in te newer version causing the issue. In such case, follow the steps to update `login.css`
- Get the new copy of `login.css` by configuring the login page with keycloak base theme
- View page source of login page
- Get the new login.css copy
- Update the background image
- Update the `rimrep-flux` and commit the changes
- Above PRs for reference 
