This doc explains the upgrade steps for different apps like flux, keycloak, krakend, argo, nginx ingress controller and  Oauth proxy.

## Flux
Flux is deployed via Terraform. Upgrading flux requires terraform update. 
- Update the version details in [variable.tf] (https://github.com/aodn/rimrep-terraform/blob/main/modules/flux/variables.tf#L46) to upgrade flux version.
- Follow the usual process of updating the terraform code like creating new branch from main, commit the changes, create PR, get it approved and merge to main.
- Run the (development terraform workspace)[https://app.terraform.io/app/rimrep-dms/workspaces/rimrep-development-infrastructure?organization_name=rimrep-dms] for apply the changes in development
- Check the flux pods in development cluster after upgrade. All pods should be running. Check the logs to confirm no error.
- Once after testing, move the changes to (staging)[https://app.terraform.io/app/rimrep-dms/workspaces/rimrep-staging-v2-infrastructure?organization_name=rimrep-dms]
- Perform testing in staging. Once after testing in staging, promote the changes to production.
-  Production workspace is yet to be configured. 

## Keycloak
Keycloak is running as deployment in EKS cluster using flux. It uses external postgres database. So database upgrade is not required unless a new keycloak version requires a specific database postgres version. Upgrading keycloak is simple by upgrading the docker image version in `rimrep-flux` repo.

### Steps to upgrade Keycloak
- New versions of Keycloak can be found here https://github.com/keycloak/keycloak/releases.
- Perform any pre-requisite if required based on the new version.
- Create a new branch in [rimrep-flux](https://github.com/aodn/rimrep-flux/tree/main) fir Keycloak upgrade.
- Update version details in respective environment [deployment yaml](https://github.com/aodn/rimrep-flux/blob/main/apps/development/keycloak/deployment-patch.yaml) Do not update base yaml as it will update all environments at once before testing it in development environment.
- Commit, push and create PR.
- Merge the changes after approval.
- Wait for flux to update the new committed change.
- Check Keycloak changes after flux update.
**Note :** Refer docs/guidelines/keycloak-management.md when there is an issue in loading background image in Keycloak login page

## Krakend
Krakend uses customized docker image as it requires AWS plugin for S3 bucket access. As it is using customized docker image, Krakend image needs to be created first before its upgrade.

### Steps to upgrade Krakend
- Krakend image creation is available in the repo [rimrep-krakend-plugin](https://github.com/aodn/rimrep-krakend-plugin)
- Krakend version is controlled in [workflows yaml](https://github.com/aodn/rimrep-krakend-plugin/blob/main/.github/workflows/build.yaml#L12).
- Follow the usual process by creating a new branch to update the Krakend version
- Pipeline will take care of creating the Krakend image with AWS plugn and push it to ECR repo in respective AWS account.
- Image automation controller is enabled in flux so no need to update the new image tag after image creation. Flux will take care of patching the Krakend version.

## Argo
Argo deployment is managed as Helm release. Upgrading argo needed Helm chart version to in helm release to be updated. Helm release for argo are maintained in `rimrep-flux` repo. When argo needs upgrade, update the version in argo-values.yaml file in `apps\{environment}\argo\argo-values.yaml` with new version and commit the changes. If the version is not configured in argo-values.yaml, add a entry for version to overlay the version details from base yaml and commit the changes. Flux will take care of updating the helm release after committing the changes to main branch. Please ensure the changes are tested in development/test environment before proceeding to production. 

## Nginx Ingress Controller
Nginx ingress controlled is deployed as Helm release and it will be available in `/infrastructure` folder of rimrep-flux repo. It is not like other apps which are managed under `/apps` folder. It is using wildcard version and will be auto upgraded when new version is released. It is good for development and staging environment. For production, using wildcard opion is not advisable as auto upgrade may cause breaking changes. So when deploying for production, use patch yaml to update the helm chart version to specific version details.

## Oauth proxy
Oauth proxy is similar to Nginx ingress controller with wild card for version. Patch file is available here `apps/{environment}/oauth2-proxy-keycloak/oauth2-proxy-values.yaml`. For production, use some specific version of helm chart for the app not to be updated with any breaking changes. 
