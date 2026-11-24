## Autoscaling with HPA

Folder: `yaml-definitions/local-ttt-app-hpa-deploy`

An HPA (Horizontal Pod Autoscaler) watches the app's CPU. When the pods get busy, it adds more. When it's quiet, it removes them.

Goal:
- Minimum: **2** replicas, maximum: **10**
- Only the app scales. Mongo stays at 1 replica.

### Step 1: Install metrics-server

The HPA needs CPU numbers from metrics-server. On Docker Desktop it needs one extra flag:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl patch deployment metrics-server -n kube-system --type=json -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'

kubectl top pods
```


### Step 2: Change `ttt-deploy.yml`

- Removed `replicas: 3`, because the HPA sets the number now
- Added CPU `resources`, because the HPA needs a CPU request to work out the percentage

```yaml
          resources:
            requests:
              cpu: 100m
            limits:
              cpu: 200m
```
- 100m = 0.1 of a core
- 200m = 0.2 of a core

### Step 3: Create `ttt-hpa.yml`

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ttt-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ttt-app-deployment   
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

Above 50% average CPU, it adds pods. Below, it removes them.

### Step 4: Deploy

```bash
kubectl apply -f ttt-deploy.yml
kubectl apply -f ttt-hpa.yml
kubectl get hpa
kubectl get pods
```


### Step 5: Load test

Open a new terminal and send traffic with Apache Bench (through Docker, so no install needed):

```bash
docker run --rm httpd:2.4 ab -n 100000 -c 100 http://host.docker.internal:30001/
```

If CPU goes above 50%, so the HPA adds pods, up to 10.


### Step 6: Scale down

After the test, the HPA waits about 5 minutes, then goes back to 2 pods.


### Clean up

```bash
kubectl delete -f ttt-hpa.yml
```

### Problem and fix

| Problem | Fix |
|---|---|
| metrics-server showed `0/1` and `kubectl top pods` failed | Added the `--kubelet-insecure-tls` flag |
