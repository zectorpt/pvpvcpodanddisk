# Create a pod with a pv in a disk
az aks show --resource-group onenode --name onenode --query nodeResourceGroup -o tsv

MC_onenode_onenode_switzerlandnorth

az disk create \
  --resource-group MC_onenode_onenode_switzerlandnorth \
  --name myAKSDisk \
  --size-gb 25 \
  --query id --output tsv
  

/subscriptions/a67cbe10-1391-446f-a40e-c0e41d4c7538/resourceGroups/MC_onenode_onenode_switzerlandnorth/providers/Microsoft.Compute/disks/myAKSDisk


pv-azuredisk.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.kubernetes.io/provisioned-by: disk.csi.azure.com
  name: pv-azuredisk
spec:
  capacity:
    storage: 25Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: managed-csi
  csi:
    driver: disk.csi.azure.com
    volumeHandle: /subscriptions/a67cbe10-1391-446f-a40e-c0e41d4c7538/resourceGroups/MC_onenode_onenode_switzerlandnorth/providers/Microsoft.Compute/disks/myAKSDisk
    volumeAttributes:
      fsType: ext4
	  
	  
	  
	 
pvc-azuredisk.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-azuredisk
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 25Gi
  volumeName: pv-azuredisk
  storageClassName: managed-csi
  
kubectl apply -f pv-azuredisk.yaml
kubectl apply -f pvc-azuredisk.yaml


azure-disk-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  nodeSelector:
    kubernetes.io/os: linux
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    name: mypod
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
      - name: azure
        mountPath: /usr/share/nginx/html
  volumes:
    - name: azure
      persistentVolumeClaim:
        claimName: pvc-azuredisk
		
kubectl apply -f azure-disk-pod.yaml

kubectl label pod mypod app=mypod
kubectl expose pod mypod --type=LoadBalancer --port=80 --target-port=80 --name=my-service005

kubectl exec --stdin --tty mypod -- sh


kubectl delete -f pvc-azuredisk.yaml
kubectl delete -f pv-azuredisk.yaml
