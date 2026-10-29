# Lab 13: Persistent Storage Setup for Application Logging

## Objective
Create a PersistentVolume (PV) and a PersistentVolumeClaim (PVC) to store application logs.

## Files
- `pv.yaml` : PersistentVolume
- `pvc.yaml` : PersistentVolumeClaim

## Specifications

| Item | PV | PVC |
|------|----|-----|
| Name | ivolvepv | ivolvepvc |
| Size | 1Gi | 1Gi |
| Access mode | ReadWriteMany | ReadWriteMany |
| Storage type | hostPath (`/mnt/app-logs`) | - |
| Reclaim policy | Retain | - |
| StorageClass | none | none |

> Kubernetes names must be lowercase, so `ivolvePV` / `ivolvePVC` are written as `ivolvepv` / `ivolvepvc`.

## Steps

1. Create the namespace (if it does not exist):
   ```bash
   kubectl create namespace ivolve
   ```

2. Apply the PV and the PVC:
   ```bash
   kubectl apply -f pv.yaml
   kubectl apply -f pvc.yaml
   ```

3. Verify that both are `Bound`:
   ```bash
   kubectl get pv
   kubectl get pvc -n ivolve
   ```

   Expected output:
   ```
   NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
   ivolvepv   1Gi        RWX            Retain           Bound    ivolve/ivolvepvc

   NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS
   ivolvepvc   Bound    ivolvepv   1Gi        RWX
   ```

4. Check the details:
   ```bash
   kubectl describe pv ivolvepv
   kubectl describe pvc ivolvepvc -n ivolve
   ```

## Screenshots
Add your screenshots here:

- `kubectl apply` output
- `kubectl get pv,pvc` showing `Bound`

## Notes
- `storageClassName: ""` disables dynamic provisioning, so no StorageClass is used.
- `volumeName: ivolvepv` makes the PVC bind to this specific PV.
- `Retain` keeps the data on the node even if the PVC is deleted.
