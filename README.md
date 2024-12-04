# Create a pod with a pv in a disk
az aks show --resource-group onenode --name onenode --query nodeResourceGroup -o tsv

MC_onenode_onenode_switzerlandnorth

az disk create --resource-group MC_onenode_onenode_switzerlandnorth --name myAKSDisk --size-gb 25 --query id --output tsv
  

/subscriptions/[SUB]/resourceGroups/MC_onenode_onenode_switzerlandnorth/providers/Microsoft.Compute/disks/myAKSDisk
  
kubectl apply -f pv-azuredisk.yaml
kubectl apply -f pvc-azuredisk.yaml		
kubectl apply -f azure-disk-pod.yaml

kubectl label pod mypod app=mypod
kubectl expose pod mypod --type=LoadBalancer --port=80 --target-port=80 --name=my-service001

kubectl exec --stdin --tty mypod -- sh

# To delete everything
kubectl delete -f azure-disk-pod.yaml <br>
kubectl delete -f pvc-azuredisk.yaml <br>
kubectl delete -f pv-azuredisk.yaml <br>
