# Lab 13: Persistent Storage Setup for Application Logging

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
