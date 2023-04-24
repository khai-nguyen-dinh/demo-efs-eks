
## Deploy EFS CSI Driver

We are going to deploy the driver using the stable release:

```
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.3"
```

Verify pods have been deployed:
```
kubectl get pods -n kube-system
```

Should return new pods with csi driver:

Output: 
{{< output >}}
NAME                       READY   STATUS    RESTARTS   AGE
efs-csi-node-8hgpt         3/3     Running   0          6h11m
efs-csi-node-d7r47         3/3     Running   0          6h11m
efs-csi-node-fs49j         3/3     Running   0          6h11m
{{< /output >}}


## Create Persistent Volume
Next we will deploy a persistent volume using the EFS created. 
We need to update this manifest with the EFS ID created:
```
sed -i "s/EFS_VOLUME_ID/$FILE_SYSTEM_ID/g" efs-pvc.yaml
```

And then apply:
```
kubectl apply -f efs-pvc.yaml
```

Next, check if a PVC resource was created. The output from the command should look similar to what is shown below, with the **STATUS** field set to **Bound**.
```
kubectl get pvc -n storage
```

Output: 
{{< output >}}
NAME                STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
efs-storage-claim   Bound    efs-pvc   5Gi        RWX            efs-sc         4s
{{< /output >}}


A PV corresponding to the above PVC is dynamically created. Check its status with the following command.
```
kubectl get pv
```

Output: 
{{< output >}}
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   REASON   AGE
efs-pvc   5Gi        RWX            Retain           Bound    storage/efs-storage-claim   efs-sc                  20s
{{< /output >}}


## Create sample application
We will create sample application with multiple mount.
Create 3 pod write to one file, 1 pod read:
```
# create read 1 pod
kubectl apply -f efs-reader.yaml
# create 3 write pods
kubectl apply -f efs-writer.yaml
```
Or you can create with helm:

```
# create read 1 pod
helm install read-pod demo --set customCMD.myCustomCommand="tail -f out.txt" --set efsID="<EFS ID>" -n storage

# create 3 write pods
helm install read-pod demo --set customCMD.myCustomCommand: "/bin/sh -c while true; do echo $HOSTNAME - $(date -u) >> /shared/out.txt; sleep 5; done"  --set efsID="<EFS ID>" -n storage
```
