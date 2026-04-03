# DevOps Lab: Docker + Nginx + Kubernetes

## 👨‍🎓 ИУ12_11М: Литовченко А.О.

---

## 📌 Описание проекта

В рамках лабораторной работы реализовано веб-приложение на Flask с полным циклом развертывания:

* Docker (контейнеризация)
* Docker Compose (оркестрация)
* Nginx (reverse proxy)
* Kubernetes (Minikube)
* ConfigMap (конфигурация)
* Ingress (доменный доступ)
* Rolling Update (обновление без downtime)

---

## 📁 Структура проекта

```
devops-lab/
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
├── nginx/
│   └── nginx.conf
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── ingress.yaml
├── docker-compose.yml
├── README.md
└── report.md
```

---

## 🚀 Приложение

Flask API с двумя endpoint:

### `/`

```json
{
  "service": "devops-lab",
  "hostname": "container-id",
  "version": "1.0.0"
}
```

### `/health`

```json
{
  "status": "ok"
}
```

---

## 🐳 Docker

### Сборка образа

```bash
docker build -t devops-lab:v1 ./app
```

### Запуск

```bash
docker run -d -p 5000:5000 devops-lab:v1
```

---

## 🧩 Docker Compose + Nginx

### Запуск

```bash
docker compose up -d --build
```

### Проверка

```bash
curl http://localhost
curl http://localhost/health
```

### Что делает Nginx

* reverse proxy к Flask
* пробрасывает заголовки
* обрабатывает `/health`
* отдаёт ошибки (50x)

---

## ☸️ Kubernetes (Minikube)

### Запуск кластера

```bash
minikube start
```

### Загрузка образа

```bash
minikube image load devops-lab:v1
```

### Деплой

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### Проверка

```bash
kubectl get pods
kubectl get svc
```

---

# ⭐ БОНУСЫ

---

## 🔹 Бонус 1: ConfigMap

Вынесли `APP_VERSION` из Deployment.

### `configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: devops-lab-config
data:
  APP_VERSION: "1.0.0"
```

### В Deployment

```yaml
env:
  - name: APP_VERSION
    valueFrom:
      configMapKeyRef:
        name: devops-lab-config
        key: APP_VERSION
```

### Обновление версии

```bash
kubectl edit configmap devops-lab-config
kubectl rollout restart deployment devops-lab
```

---

## 🔹 Бонус 2: Ingress

Доступ по доменному имени вместо NodePort.

### Включение

```bash
minikube addons enable ingress
```

### `ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: devops-lab-ingress
spec:
  rules:
    - host: devops-lab.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: devops-lab-service
                port:
                  number: 80
```

### Hosts

```bash
echo "$(minikube ip) devops-lab.local" | sudo tee -a /etc/hosts
```

### Проверка

```bash
curl http://devops-lab.local
```

---

## 🔹 Бонус 3: Rolling Update

Обновление без остановки сервиса.

### Сборка нового образа

```bash
eval $(minikube docker-env)
docker build -t devops-lab:v2 ./app
```

### Обновление Deployment

```yaml
image: devops-lab:v2
```

### Применение

```bash
kubectl apply -f k8s/deployment.yaml
```

### Наблюдение

```bash
kubectl rollout status deployment/devops-lab
kubectl get pods -w
```

### Откат

```bash
kubectl rollout undo deployment/devops-lab
```

---

## 🧠 Полезные команды

### Docker

```bash
docker ps
docker logs <container>
docker compose down
```

### Kubernetes

```bash
kubectl get all
kubectl describe pod <pod>
kubectl logs <pod>
kubectl rollout status deployment/devops-lab
```

---

## ✅ Результат

В ходе работы реализовано:

* контейнеризация приложения
* reverse proxy через Nginx
* запуск через Docker Compose
* деплой в Kubernetes
* конфигурация через ConfigMap
* доступ через Ingress
* rolling update без downtime
