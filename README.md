# kubernetes-demo

apiVersion: v1
kind: Namespace
metadata:
  name: demo-webapplication
---
apiVersion: v1
kind: Namespace
metadata:
  name: prod-webapplication
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: demo-kubernetes-demo-proxy
  namespace: demo-webapplication
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: prod-kubernetes-demo-proxy
  namespace: prod-webapplication
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: Service
metadata:
  name: demo-webapplication
  namespace: demo-webapplication
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: webapplication
---
apiVersion: v1
kind: Service
metadata:
  name: prod-webapplication
  namespace: prod-webapplication
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: webapplication
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    monitoring: "true"
  labels:
    dummy: word
  name: demo-nginx-proxy
  namespace: demo-webapplication
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapplication
  template:
    metadata:
      labels:
        app: webapplication
    spec:
      containers:
      - image: nginx
        name: webapplication
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: volume-test
      volumes:
      - name: volume-test
        persistentVolumeClaim:
          claimName: demo-kubernetes-demo-proxy
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    monitoring: "true"
  labels:
    dummy: word
  name: prod-nginx-proxy
  namespace: prod-webapplication
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapplication
  template:
    metadata:
      labels:
        app: webapplication
    spec:
      containers:
      - image: nginx
        name: webapplication
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: volume-test
      volumes:
      - name: volume-test
        persistentVolumeClaim:
          claimName: prod-kubernetes-demo-proxy
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernet.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: demo-ingress
  namespace: demo-webapplication
spec:
  rules:
  - host: demo.fileserver.local
    http:
      paths:
      - backend:
          serviceName: demo-webapplication
          servicePort: 80
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernet.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: prod-ingress
  namespace: prod-webapplication
spec:
  rules:
  - host: prod.fileserver.local
    http:
      paths:
      - backend:
          serviceName: prod-webapplication
          servicePort: 80
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-kubernetes-demo-proxy
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  local:
    path: /data/containers/kubernetes/test01
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - fileserver
  storageClassName: kubernetes-demo-proxy
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: prod-kubernetes-demo-proxy
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  local:
    path: /data/containers/kubernetes/test01
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - fileserver
  storageClassName: kubernetes-demo-proxy
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-kubernetes-demo-proxy
  namespace: demo-webapplication
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: kubernetes-demo-proxy
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prod-kubernetes-demo-proxy
  namespace: prod-webapplication
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: kubernetes-demo-proxy
