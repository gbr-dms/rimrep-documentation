## EKS upgrade steps
Below are the steps for upgrading EKS version. It is a high level steps to cover the upgrade process. Actual upgrade process might slightly differ based on the version you are upgrading. Read the vesion release notes and eks upgrade doc before starting the upgrade.

### Pre-requisite
- EKS can be upgraded one minor version at a time. For example, EKS can be upgraded from 1.26 to 1.27 but not to 1.28 directly. To upgrade to 1.28, upgrade first to 1.27 and then to 1.28.
- Read the release notes https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions-standard.html and do necessary pre-requisite tasks if required for each upgrade if it is not part of the Terraform code.
- Check the add-on versions of aws vpc cni, core DNS and kube-proxy is compatable with new EKS version. Terraform code will upgrade the add-on to the latest version automatically. Rarely add-on updates might fail. If failed, fix it using `kubectl` commands.
- Kubernetes minor version of nodes in the cluster should match control plane's version. If not, node needs to be upgrade to match the contraol plane's version before starting the upgrade.
- Refer the doc https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html once before starting the upgrade process.
  
### EKS upgrade steps
- Clone to [rimrep-terraform](https://github.com/aodn/rimrep-terraform) repo
- Checkout a new branch for version upgrade
- Change the `cluster_version` to the version you want to upgrade in `eks.auto.tfvars` of respective environment. For example,
  - Development environment - **development/infrastructure/eks.auto.tfvars**
  - Staging-v2 environment - **staging-new/infrastructure/eks.auto.tfvars**
- Run plan to verify the changes
- Commit and push the changes after successful plan run
- Create pull request and merge the changes to `main` branch after reviewer approval
- Terraform workspace automatically run plan after merge, check plan output for the merge commit
- Run apply
- Monitor the upgrade Usually takes 15+ minutes to complete the upgrade.
- Check the status of the pods and nodes after the upgrade
- In case of issues such as pod not coming to running state or restarting frequently, describe the pod status or check the logs to fix the issue.
- Next step is to upgrade the nodes after control plane upgrade

### Node upgrade steps
- Nodes to be upgrade after EKS upgrade to match control plane minor version
- It has to be done manually as the nodes will be drained and deleted one my one to avoid distruptions to services.
- Drain the nodes one by one. It can be done using k9s tool or kubectl
  - K9s - Go to node, select a node, press r (drain option), select ignore daemonsets, delete local data, force option, select `OK` and enter.
  - Kubectl - Get the node name, `kubectl drain --ignore-daemonsets --delete-local-data <node name> `
- Once after draining, wait for new node will come up check for kubernetes version. It should match the new EKS version major and minor.
- In case of any issues, describe the new node and investigate the issue.
- Delete the drained node.
  - K9s - Select the drained node, press ctlr+d (delete option), select `OK` and enter.
  - Kubectl - Get the node name, `kubectl delete <node name>`
- Repeat the process for all nodes to match node kubernete version with control plane version
- In case of issues with nodes not coming to running state, describe the node status or check the logs to fix the issue.

