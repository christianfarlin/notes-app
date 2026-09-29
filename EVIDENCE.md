# Lab 5 Evidence - Christian Farlin

## Successful GitHub Actions Run

The GitHub Actions workflow successfully built and published the application image to Docker Hub.

![Successful GitHub Actions Run](evidence/Successful%20GitHub%20Actions%20Run.png)

## Docker Hub Multi-Architecture Image

The published Docker Hub image is multi-platform and supports both `linux/amd64` and `linux/arm64`.

![Docker Hub Multi-Architecture Image](evidence/Docker%20Hub%20Multi-Architecture%20Image.png)

## Kubernetes Resources

Command:

```text
kubectl get all,pvc
```

Output:

```text
NAME                         READY   STATUS    RESTARTS   AGE
pod/db-6c5c8947cd-v7gxv     1/1     Running   0          6m12s
pod/web-64ff8d68ff-vqt6v    1/1     Running   0          13s
pod/web-64ff8d68ff-xzq8f    1/1     Running   0          13s

NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/db    ClusterIP   10.96.72.97    <none>        5432/TCP   6m12s
service/web   ClusterIP   10.96.180.80   <none>        80/TCP     13s

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db    1/1     1            1           6m12s
deployment.apps/web   2/2     2            2           13s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/db-6c5c8947cd    1         1         1       6m12s
replicaset.apps/web-64ff8d68ff   2         2         2       13s

NAME                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/db-data   Bound    pvc-62443c1b-a73b-410c-a947-08dae0a3be1a   1Gi        RWO            standard       <unset>                 6m12s
```

## Experiment 2: Data Persistence

After creating the note `"I should survive a pod deletion"` and deleting the database pod, the replacement database pod started successfully.

Command:

```text
curl http://localhost:8000/notes
```

Output:

```json
[{"body":"hello from kubernetes","created_at":"2026-09-29T01:06:17.332474+00:00","id":1},{"body":"I should survive a pod deletion","created_at":"2026-09-29T01:15:02.006064+00:00","id":2}]
```

The note persisted after the database pod was deleted and recreated because the database data is stored on the persistent volume associated with the `db-data` PVC.

## Experiment 3: Load Balancing

The web deployment was scaled to four replicas and multiple requests were sent to the web Service from inside the Kubernetes cluster.

Output:

```json
{"message":"Hello from the updated notes app!","served_by":"web-64ff8d68ff-9t7bf","service":"notes-app"}
{"message":"Hello from the updated notes app!","served_by":"web-64ff8d68ff-v8684","service":"notes-app"}
{"message":"Hello from the updated notes app!","served_by":"web-64ff8d68ff-xpkfj","service":"notes-app"}
{"message":"Hello from the updated notes app!","served_by":"web-64ff8d68ff-xpkfj","service":"notes-app"}
```

The different `served_by` values demonstrate that the Kubernetes Service distributed requests across multiple web pods.

## Experiment 4: Rolling Update and Rollout History

The web Deployment was updated to the immutable image tag `christianj2026/notes-app:sha-55dd55d`.

Command:

```text
kubectl rollout history deployment/web
```

Output after the rolling update:

```text
deployment.apps/web
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

The rolling update completed successfully. The Deployment was subsequently rolled back using `kubectl rollout undo deployment/web`.