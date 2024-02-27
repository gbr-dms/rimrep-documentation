# Previewing service changes in dev env

This document describes how you can make changes to a deployed service like Pygeoapi or STAC and preview that change in the development environment before you open a PR to main branch.  It’s written using pygeoapi as an example but the same steps apply when deploying stac.

Push your feature branch
Tag your feature branch commit (for new commits you want to preview just increment the semver postfix 0.0.N):

```
git tag deploy-from-feature-branch-0.0.1
```

Push your tag:
```
git push --tags
```
​
Wait for you build to finish https://github.com/aodn/rimrep-pygeoapi/actions/workflows/main.yml
Manually run the deployment for the new build https://github.com/aodn/rimrep-pygeoapi/actions/workflows/containers.yml

Get the new container image tag

Using the command line:

```
aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 058939800614.dkr.ecr.ap-southeast-2.amazonaws.com/
aws ecr list-images \
--registry-id 058939800614 \
--page-size 400 \
--repository-name reefdata/pygeoapi \
--filter tagStatus=TAGGED \
--query 'imageIds[*].imageTag' | grep deploy-from-feature-branch
```

Or by referring to the build output.

In your terminal, connect to the cluster:
```
export AWS_PROFILE=rimrep-dev-admin && kubectx arn:aws:eks:ap-southeast-2:058939800614:cluster/rimrep-development-eks
```
​
Check the current deployed version:

```
kubectl get deployment.apps/pygeoapi -n pygeoapi -o=jsonpath='{.spec.template.spec.containers[*].image}'
```

Suspend Flux kustomization so it doesn't revert your changes:
```
flux suspend kustomization apps
```
​
Update the current deployed version:
```
kubectl set image deployment.apps/pygeoapi pygeoapi=058939800614.dkr.ecr.ap-southeast-2.amazonaws.com/reefdata/pygeoapi:deploy-from-feature-branch-7302cf21-1706495243
```
​
Check the current deployed version to make sure the change was made:
```
kubectl get deployment.apps/pygeoapi -n pygeoapi -o=jsonpath='{.spec.template.spec.containers[*].image}'
```

Watch the new pod get deployed and the old one terminated:
```
kubectl get pods -n pygeoapi --watch
```
​
Check your deployment.

When you are done and wish to revert back then just resume flux:
```
flux resume kustomization apps
```