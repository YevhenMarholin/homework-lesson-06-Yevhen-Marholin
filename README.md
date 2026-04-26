# homework-lesson-06-Yevhen-Marholin

## 1. Push image у Docker Hub

Було зібрано Docker image для `apps/course-app`.

```bash
docker build -t course-app .
```

Образ було затеговано для Docker Hub:

```bash
docker tag course-app yevhen_marholin/course-app:latest
```

Образ було завантажено у Docker Hub:

```bash
docker push yevhen_marholin/course-app:latest
```

---

## 2. Deployment manifest

Було створено файл `deployment.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: course-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: course-app
  template:
    metadata:
      labels:
        app: course-app
    spec:
      containers:
        - name: course-app
          image: yevhen_marholin/course-app:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
            - name: APP_STORE
              value: redis
            - name: APP_REDIS_URL
              value: redis://redis:6379/0
```

---

## 3. Service manifest

Було створено файл `service.yaml`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: course-app-service
spec:
  type: NodePort
  selector:
    app: course-app
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30080
```

---

## 4. Redis manifest

Для роботи застосунку з Redis було створено файл `redis.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7-alpine
          ports:
            - containerPort: 6379
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
```

---

## 5. Deploy у Rancher Desktop cluster

```bash
kubectl apply -f redis.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Перевірка ресурсів:

```bash
kubectl get pods
kubectl get deployments
kubectl get services
```

---

## 6. Перевірка застосунку

Service має тип `NodePort`, тому застосунок доступний на порту `30080`.

```bash
curl http://localhost:30080
curl http://localhost:30080/healthz
```

Або у браузері:

```text
http://localhost:30080
http://localhost:30080/healthz
```

---

## 7. Зміна кількості реплік

У файлі `deployment.yaml` було змінено кількість реплік:

```yaml
replicas: 3
```

Після цього deployment було застосовано повторно:

```bash
kubectl apply -f deployment.yaml
```

Перевірка процесу оновлення:

```bash
kubectl rollout status deployment/course-app
```

Перевірка pod-ів:

```bash
kubectl get pods
```

---

## Висновок

У межах домашнього завдання було:

- зібрано Docker image для `apps/course-app`
- завантажено image у Docker Hub
- описано Kubernetes `Deployment`
- описано Kubernetes `Service` з типом `NodePort`
- додано Redis як зовнішнє сховище для застосунку
- задеплоєно ресурси у Rancher Desktop Kubernetes cluster
- змінено кількість реплік у `deployment.yaml`
- перевірено процес оновлення через `kubectl rollout status deployment/course-app`