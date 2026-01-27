
## 1. Config Kube Service
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: demo
  name: service-example
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: service-example
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: service-example
    spec:
      containers:
        - image: registry-example.co.id/dev/service-example
          imagePullPolicy: Always
          name: service-example
          ports:
            - containerPort: 80
          envFrom:
            - configMapRef:
                name: service-example-env
      imagePullSecrets:
        - name: regcred
```

## 2. Kube Config Map

```yml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-example-env
  namespace: demo
data:
  PORT: "8011"
  TIMEZONE: "Asia/Jakarta"
  TIMEOUT_GRACEFUL_SHUTDOWN: "30"

  MYSQL_READ_HOST: "localhost"
  MYSQL_READ_USER: "root"
  MYSQL_READ_PASS: "root"
  MYSQL_READ_DBNAME: "service_example"
  MYSQL_READ_PORT: "4000"
  MYSQL_READ_USE_TLS: "true"
```

## 3. Kube Config Network

```yml
apiVersion: v1
kind: Service
metadata:
 name: service-example
 namespace: samb
spec:
 selector:
   app.kubernetes.io/name: service-example
 ports:
   - name: http
     protocol: TCP
     port: 80
     targetPort: 80
 type: NodePort
 clusterIP: 10.111.245.111

```

## 4. Kube Config Cronjob

```yml

apiVersion: batch/v1
kind: CronJob
metadata:
 name: cronjob-example
 namespace: cronjob
spec:
 schedule: "0 */5 * * *"
 jobTemplate:
   spec:
     template:
       spec:
         containers:
         - name: curl-post
           image: curlimages/curl:8.17.0
           command:
             - sh
             - -c
           args:
             - |
               curl -X POST "http://10.98.205.115/v1/example/create" \
               -H "Content-Type: application/json"
         restartPolicy: OnFailure
     backoffLimit: 3
 successfulJobsHistoryLimit: 3
 failedJobsHistoryLimit: 1

```
