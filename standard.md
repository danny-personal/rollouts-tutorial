```
gcloud beta container --project "rollouts-sandbox" clusters create "rollout-a" --zone "asia-northeast1-c" --no-enable-basic-auth --cluster-version "1.24.9-gke.2000" --release-channel "regular" --machine-type "e2-standard-4" --image-type "COS_CONTAINERD" --disk-type "pd-balanced" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/cloud-platform" --max-pods-per-node "110" --num-nodes "1" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/rollouts-sandbox/global/networks/default" --subnetwork "projects/rollouts-sandbox/regions/asia-northeast1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --node-locations "asia-northeast1-c"
```

```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

```
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x ./kubectl-argo-rollouts-linux-amd64 
mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
kubectl argo rollouts version
```

```
root@d4c0a80bd057:/supay-tools/scripts/environments# kubectl argo rollouts version
kubectl-argo-rollouts: v1.4.0+e40c9fe
  BuildDate: 2023-01-09T20:20:38Z
  GitCommit: e40c9fe8a2f7fee9d8ee1c56b4c6c7b983fce135
  GitTreeState: clean
  GoVersion: go1.19.4
  Compiler: gc
  Platform: linux/amd64
root@d4c0a80bd057:/supay-tools/scripts/environments#
```

```
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/rollout.yaml
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/service.yaml
kubectl argo rollouts get rollout rollouts-demo --watch
```

```
Name:            rollouts-demo
Namespace:       default
Status:          ✔ Healthy
Strategy:        Canary
  Step:          8/8
  SetWeight:     100
  ActualWeight:  100
Images:          argoproj/rollouts-demo:blue (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       5
  Ready:         5
  Available:     5

NAME                                       KIND        STATUS     AGE  INFO
⟳ rollouts-demo                            Rollout     ✔ Healthy  14m  
└──# revision:1                                                        
   └──⧉ rollouts-demo-687d76d795           ReplicaSet  ✔ Healthy  14m  stable
      ├──□ rollouts-demo-687d76d795-6l8h6  Pod         ✔ Running  14m  ready:1/1
      ├──□ rollouts-demo-687d76d795-dgdcp  Pod         ✔ Running  14m  ready:1/1
      ├──□ rollouts-demo-687d76d795-lzlcc  Pod         ✔ Running  14m  ready:1/1
      ├──□ rollouts-demo-687d76d795-q9fn9  Pod         ✔ Running  14m  ready:1/1
      └──□ rollouts-demo-687d76d795-xp6d8  Pod         ✔ Running  14m  ready:1/1
```

```
Name:            rollouts-demo
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
  Step:          1/8
  SetWeight:     20
  ActualWeight:  20
Images:          argoproj/rollouts-demo:blue (stable)
                 argoproj/rollouts-demo:yellow (canary)
Replicas:
  Desired:       5
  Current:       5
  Updated:       1
  Ready:         5
  Available:     5

NAME                                       KIND        STATUS     AGE    INFO
⟳ rollouts-demo                            Rollout     ॥ Paused   20m    
├──# revision:2                                                          
│  └──⧉ rollouts-demo-6cf78c66c5           ReplicaSet  ✔ Healthy  2m25s  canary
│     └──□ rollouts-demo-6cf78c66c5-p4v48  Pod         ✔ Running  2m24s  ready:1/1
└──# revision:1                                                          
   └──⧉ rollouts-demo-687d76d795           ReplicaSet  ✔ Healthy  20m    stable
      ├──□ rollouts-demo-687d76d795-6l8h6  Pod         ✔ Running  20m    ready:1/1
      ├──□ rollouts-demo-687d76d795-dgdcp  Pod         ✔ Running  20m    ready:1/1
      ├──□ rollouts-demo-687d76d795-lzlcc  Pod         ✔ Running  20m    ready:1/1
      └──□ rollouts-demo-687d76d795-xp6d8  Pod         ✔ Running  20m    ready:1/1
```

```
kubectl argo rollouts promote rollouts-demo
```

```
Name:            rollouts-demo
Namespace:       default
Status:          ✔ Healthy
Strategy:        Canary
  Step:          8/8
  SetWeight:     100
  ActualWeight:  100
Images:          argoproj/rollouts-demo:yellow (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       5
  Ready:         5
  Available:     5

NAME                                       KIND        STATUS        AGE    INFO
⟳ rollouts-demo                            Rollout     ✔ Healthy     23m    
├──# revision:2                                                             
│  └──⧉ rollouts-demo-6cf78c66c5           ReplicaSet  ✔ Healthy     5m14s  stable
│     ├──□ rollouts-demo-6cf78c66c5-p4v48  Pod         ✔ Running     5m13s  ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-p4t87  Pod         ✔ Running     2m8s   ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-k6p96  Pod         ✔ Running     116s   ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-qp8zc  Pod         ✔ Running     105s   ready:1/1
│     └──□ rollouts-demo-6cf78c66c5-969bn  Pod         ✔ Running     94s    ready:1/1
└──# revision:1                                                             
   └──⧉ rollouts-demo-687d76d795           ReplicaSet  • ScaledDown  23m   
```

```
kubectl argo rollouts set image rollouts-demo \
  rollouts-demo=argoproj/rollouts-demo:red
```

```
Name:            rollouts-demo
Namespace:       default
Status:          ॥ Paused
Message:         CanaryPauseStep
Strategy:        Canary
  Step:          1/8
  SetWeight:     20
  ActualWeight:  20
Images:          argoproj/rollouts-demo:red (canary)
                 argoproj/rollouts-demo:yellow (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       1
  Ready:         5
  Available:     5

NAME                                       KIND        STATUS        AGE    INFO
⟳ rollouts-demo                            Rollout     ॥ Paused      25m    
├──# revision:3                                                             
│  └──⧉ rollouts-demo-5747959bdb           ReplicaSet  ✔ Healthy     81s    canary
│     └──□ rollouts-demo-5747959bdb-xhntq  Pod         ✔ Running     81s    ready:1/1
├──# revision:2                                                             
│  └──⧉ rollouts-demo-6cf78c66c5           ReplicaSet  ✔ Healthy     7m30s  stable
│     ├──□ rollouts-demo-6cf78c66c5-p4v48  Pod         ✔ Running     7m29s  ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-k6p96  Pod         ✔ Running     4m12s  ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-qp8zc  Pod         ✔ Running     4m1s   ready:1/1
│     └──□ rollouts-demo-6cf78c66c5-969bn  Pod         ✔ Running     3m50s  ready:1/1
└──# revision:1                                                             
   └──⧉ rollouts-demo-687d76d795           ReplicaSet  • ScaledDown  25m    
```

```
kubectl argo rollouts abort rollouts-demo
```

```
Name:            rollouts-demo
Namespace:       default
Status:          ✖ Degraded
Message:         RolloutAborted: Rollout aborted update to revision 3
Strategy:        Canary
  Step:          0/8
  SetWeight:     0
  ActualWeight:  0
Images:          argoproj/rollouts-demo:yellow (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       0
  Ready:         5
  Available:     5

NAME                                       KIND        STATUS        AGE    INFO
⟳ rollouts-demo                            Rollout     ✖ Degraded    26m    
├──# revision:3                                                             
│  └──⧉ rollouts-demo-5747959bdb           ReplicaSet  • ScaledDown  2m9s   canary
├──# revision:2                                                             
│  └──⧉ rollouts-demo-6cf78c66c5           ReplicaSet  ✔ Healthy     8m18s  stable
│     ├──□ rollouts-demo-6cf78c66c5-p4v48  Pod         ✔ Running     8m17s  ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-k6p96  Pod         ✔ Running     5m     ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-qp8zc  Pod         ✔ Running     4m49s  ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-969bn  Pod         ✔ Running     4m38s  ready:1/1
│     └──□ rollouts-demo-6cf78c66c5-kh9tq  Pod         ✔ Running     17s    ready:1/1
└──# revision:1                                                             
   └──⧉ rollouts-demo-687d76d795           ReplicaSet  • ScaledDown  26m    
```

```
kubectl argo rollouts set image rollouts-demo \
  rollouts-demo=argoproj/rollouts-demo:yellow
```

```
Status:          ✔ Healthy
Strategy:        Canary
  Step:          8/8
  SetWeight:     100
  ActualWeight:  100
Images:          argoproj/rollouts-demo:yellow (stable)
Replicas:
  Desired:       5
  Current:       5
  Updated:       5
  Ready:         5
  Available:     5

NAME                                       KIND        STATUS        AGE    INFO
⟳ rollouts-demo                            Rollout     ✔ Healthy     31m    
├──# revision:4                                                             
│  └──⧉ rollouts-demo-6cf78c66c5           ReplicaSet  ✔ Healthy     12m    stable
│     ├──□ rollouts-demo-6cf78c66c5-p4v48  Pod         ✔ Running     12m    ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-k6p96  Pod         ✔ Running     9m41s  ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-qp8zc  Pod         ✔ Running     9m30s  ready:1/1
│     ├──□ rollouts-demo-6cf78c66c5-969bn  Pod         ✔ Running     9m19s  ready:1/1
│     └──□ rollouts-demo-6cf78c66c5-kh9tq  Pod         ✔ Running     4m58s  ready:1/1
├──# revision:3                                                             
│  └──⧉ rollouts-demo-5747959bdb           ReplicaSet  • ScaledDown  6m50s  
└──# revision:1                                                             
   └──⧉ rollouts-demo-687d76d795           ReplicaSet  • ScaledDown  31m 
```

```
cat << EOF  | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    run: my-nginx
  name: nginx-html
  namespace: hoge
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to hoge!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  namespace: hoge
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 1
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: config
          mountPath: "/usr/share/nginx/html/"
          readOnly: true
      volumes:
        - name: config
          configMap:
            name: nginx-html
            items:
            - key: "index.html"
              path: "index.html"
---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  namespace: hoge
  labels:
    run: my-nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF
```

```
cat << EOF  | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1               # Create a rollout resource
kind: Rollout
metadata:
  name: rollout-ref-deployment
  namespace: hoge
spec:
  replicas: 1
  selector:
    matchLabels:
      run: my-nginx
  workloadRef:                                 # Reference an existing Deployment using workloadRef field
    apiVersion: apps/v1
    kind: Deployment
    name: my-nginx
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: {duration: 10s}
EOF
```

``` 
cat << EOF  | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  namespace: hoge
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 0
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_FAREWELL
          value: "Such a sweet sorrow"
        ports:
        - containerPort: 80
        volumeMounts:
        - name: config
          mountPath: "/usr/share/nginx/html/"
          readOnly: true
      volumes:
        - name: config
          configMap:
            name: nginx-html
            items:
            - key: "index.html"
              path: "index.html"
EOF
```

```
kubectl argo rollouts get rollout rollout-ref-deployment -n hoge --watch
```

```
kubectl argo rollouts set image rollout-ref-deployment \
  my-nginx=nginx:1.21.4 -n hoge
```

```
kubectl argo rollouts set image rollout-ref-deployment \
  my-nginx=nginx:1.23.1 -n hoge
```

```
kubectl argo rollouts set image rollouts-demo rollouts-demo=argoproj/rollouts-demo:red
kubectl argo rollouts set image rollouts-demo rollouts-demo=argoproj/rollouts-demo:yellow
kubectl argo rollouts set image rollouts-demo rollouts-demo=argoproj/rollouts-demo:red
```
