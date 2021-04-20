# GKE Multi-cluster Ingress (MCI) Demo
A Multi-cluster Ingress (MCI) demo showing how MCI can load balance applications across regions based on client proximity and how it seamlessly routes traffic to a different region in case of failure (i.e. DR). The demo files and applications were taken from or created based on these documentation pages:
- [Provisioning Anthos clusters with Terraform](https://cloud.google.com/solutions/provisioning-anthos-clusters-with-terraform)
- [Multi-cluster Ingress](https://cloud.google.com/kubernetes-engine/docs/concepts/multi-cluster-ingress)
- [Setting up Multi-cluster Ingress](https://cloud.google.com/kubernetes-engine/docs/how-to/multi-cluster-ingress-setup)
- [Deploying Ingress across clusters](https://cloud.google.com/kubernetes-engine/docs/how-to/multi-cluster-ingress)

This demo will create a project for you, so you can start from an empty cloud shell session at [shell.cloud.google.com](shell.cloud.google.com).

## Steps
1. Go to [Google Cloud Shell](shell.cloud.google.com) and clone this repo
1. Go into the repo's folder and update the `main.tf` file with your values (`folder_id` is optional):
```sh
project_id= "unique-project-id"
billing_account = "FFFFF-FFFFF-FFFFF"
org_id = "333333333333"
[OPTIONAL] folder_id = "111111111111"
```
3. Bring up a project and the GKE clusters
```sh
terraform init && terraform apply -auto-approve
```
4. Run the `enable_mci_command` obtained from `terraform output`, which will enable MCI and set cluster1 as your "config cluster" (it looks like the command below **but you must get it from terraform**)
```sh
gcloud alpha container hub ingress enable --config-membership=projects/${var.project_id}/locations/global/memberships/${var.config_cluster_membership_name} --project=${var.project_id}
```
5. Use the `cluster2_credentials_command` obtained from `terraform output` (it looks like the command below **but you must get it from terraform**)
```sh
gcloud container clusters get-credentials ${module.gke2.name} --project=${var.project_id} --zone=${var.second_zone}
```
6. Execute the following command to deploy the namespace and application to cluster2
```sh
kubectl apply -f namespace.yaml && kubectl apply -f deploy.yaml
```
7. Use the `cluster1_credentials_command` obtained from `terraform output` (it looks like the command below **but you must get it from terraform**)
```sh
gcloud container clusters get-credentials ${module.gke1.name} --project=${var.project_id} --zone=${var.first_zone}
```
8. Execute the following command to deploy the namespace and application to cluster1
```sh
kubectl apply -f namespace.yaml && kubectl apply -f deploy.yaml
```
9. Run the following command to create a mutli-cluster service and a multi-cluster ingress (MCI) across both clusters (it works for both because you set cluster1 as your "config cluster")
```sh
kubectl apply -f mcs.yaml && kubectl apply -f mci.yaml
```
10. Query the VIP for th MCI with the following command (it won't return anything until the VIP is allocated)
```sh
echo $(kubectl get mci zone-ingress -n zoneprinter -o=jsonpath='{.status.VIP}')
```
11. Using the IP address obtained above from the multi-cluster ingress (`INGRESS_VIP`), go to `http://INGRESS_VIP/` to see the application describing which cluster (and region) you are hitting
12. You'll notice that you're routed to the cluster closest to you, so try downscaling that cluster's deployment to `0` to see how you're automatically redirected to the other cluster
```sh
kubectl scale -n zoneprinter deployment zone-ingress --replicas=0
```

## Clean-up
You have two options:
- Execute `kubectl delete -f .` on both clusters and then `terraform destroy -auto-approve` (takes 15 to 30 minutes to run)
- Shut down the Google Cloud Project (takes almost no time)