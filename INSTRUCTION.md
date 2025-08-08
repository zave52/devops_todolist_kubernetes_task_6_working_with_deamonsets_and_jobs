# Deployment Instructions for DaemonSet and CronJob

This document provides step-by-step instructions for deploying the DaemonSet and CronJob manifests to a Kubernetes cluster and validating the deployment.

## Deployment Steps

### 1. Create the mateapp namespace

First, ensure the `mateapp` namespace exists:

```bash
kubectl create namespace mateapp
```

### 2. Deploy the DaemonSet

Deploy the DaemonSet manifest:

```bash
kubectl apply -f .infrastructure/daemonset.yml
```

### 3. Deploy the CronJob

Deploy the CronJob manifest:

```bash
kubectl apply -f .infrastructure/cronjob.yml
```

### 4. Verify Deployment

Check that both resources have been created successfully:

```bash
# Check DaemonSet
kubectl get daemonset -n mateapp

# Check CronJob
kubectl get cronjob -n mateapp
```

## Validation Instructions

### Validate DaemonSet

1. **Check DaemonSet Status:**
   ```bash
   kubectl get daemonset mateapp-daemon -n mateapp
   ```
   
   Expected output should show the DaemonSet with desired/current/ready pods matching the number of nodes in your cluster.

2. **Verify DaemonSet Pods:**
   ```bash
   kubectl get pods -n mateapp -l app=mateapp-daemon
   ```
   
   You should see one pod per node in your cluster.

3. **Check DaemonSet Logs:**
   ```bash
   kubectl logs -n mateapp -l app=mateapp-daemon --tail=20
   ```
   
   Expected logs should show curl requests being made every 5 seconds to the todoapp service. You should see output similar to:
   ```
     % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
    <!DOCTYPE html>
    <html lang="en">
   ...
   ```

4. **Follow DaemonSet Logs in Real-time:**
   ```bash
   kubectl logs -n mateapp -l app=mateapp-daemon -f
   ```

### Validate CronJob

1. **Check CronJob Status:**
   ```bash
   kubectl get cronjob mateapp-cronjob -n mateapp
   ```
   
   Expected output should show the CronJob with its schedule and last schedule time.

2. **View CronJob Details:**
   ```bash
   kubectl describe cronjob mateapp-cronjob -n mateapp
   ```

3. **Check Jobs Created by CronJob:**
   ```bash
   kubectl get jobs -n mateapp
   ```
   
   You should see jobs created by the CronJob (they run every 4 minutes).

4. **Check CronJob Pods:**
   ```bash
   kubectl get pods -n mateapp | grep mateapp-cronjob
   ```

5. **Check CronJob Logs:**
   ```bash
   # Get the most recent job
   LATEST_JOB=$(kubectl get jobs -n mateapp --sort-by=.metadata.creationTimestamp | grep mateapp-cronjob | tail -n 1 | awk '{print $1}')
   
   # View logs from the latest job
   kubectl logs job/$LATEST_JOB -n mateapp
   ```
   
   Expected logs should show successful curl requests to the `/api/health` endpoint. You should see output like:
   ```
   Health OK
   ```

6. **View All CronJob Related Logs:**
   ```bash
   kubectl logs -n mateapp -l job-name --prefix=true
   ```
