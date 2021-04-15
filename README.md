# GKE Multi-cluster Ingress (MCI) Demo
A Multi-cluster Ingress (MCI) demo showing how MCI can load balance applications across regions based on client proximity and how it seamlessly routes traffic to a different region in case of failure (i.e. DR). The demo files and applications were taken from or created based on these documentation pages:
- [Provisioning Anthos clusters with Terraform](https://cloud.google.com/solutions/provisioning-anthos-clusters-with-terraform)
- [Multi-cluster Ingress](https://cloud.google.com/kubernetes-engine/docs/concepts/multi-cluster-ingress)
- [Setting up Multi-cluster Ingress](https://cloud.google.com/kubernetes-engine/docs/how-to/multi-cluster-ingress-setup)
- [Deploying Ingress across clusters](https://cloud.google.com/kubernetes-engine/docs/how-to/multi-cluster-ingress)

This demo will create a project for you, so you can start from an empty cloud shell session at [shell.cloud.google.com](shell.cloud.google.com).

## Steps
1. Update the `main.tf` file with your values (folder is optional):
    - project_id= "unique-project-id"
    - billing_account = "FFFFF-FFFFF-FFFFF"
    - org_id = "333333333333"
    - [OPTIONAL] folder_id = "111111111111" 
1. Execute:
    1. `terraform init && terraform apply -auto-approve`
    1. `mci_enable_command` obtained from `terraform output`
    1. `cluster2_credentials_command` obtained from `terraform output`
    1. `kubectl apply -f namespace.yaml && kubectl apply -f deploy.yaml` Which will deploy the namespace and application to cluster2
    1. `cluster1_credentials_command` obtained from `terraform output`
    1. `kubectl apply -f namespace.yaml && kubectl apply -f deploy.yaml` Which will deploy the namespace and application to cluster1
    1. `kubectl apply -f mcs.yaml && kubectl apply -f mci.yaml` Which will deploy the multi-cluster service and multi-cluster ingress to both clusters using the config cluster (cluster1)
    1. `echo $(kubectl get mci zone-ingress -n zoneprinter -o=jsonpath='{.status.VIP}')` and keep that IP address at hand
1. Using the IP address obtained above from the multi-cluster ingress (`INGRESS_VIP`), go to `http://INGRESS_VIP/` to see the application describing which cluster (and region) you are hitting
1. You'll notice that you're routed to the cluster closest to you, so try downscaling that cluster's deployment to `0` to see how you're automatically redirected to the other cluster
    1. Scale deployment to 0: `kubectl scale -n zoneprinter deployment zone-ingress --replicas=0`
    1. Scale back to 1: `kubectl scale -n zoneprinter deployment zone-ingress --replicas=1`

## Clean-up
You have two options:
- Execute `kubectl delete -f .` on both clusters and then `terraform destroy -auto-approve` (takes 15 to 30 minutes to run)
- Shut down the Google Cloud Project (takes almost no time)