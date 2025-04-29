# architecture-insuretech

# Task 1

# Task 2
```
minikube start --vm-driver=docker --addons=metrics-server
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
minikube service scaletestapp-service --url
minikube dashboard
locust
```
[deployment.yaml](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/deployment.yaml)

[service.yaml](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/service.yaml)

[hpa.yaml](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/hpa.yaml)

До нагрузки:
![До нагрузки](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/before_load.jpg)

Нагрузка:
![Нагрузка](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/locust.jpg)

После нагрузки:
![осле нагрузки](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/after_load.jpg)



# Task 6
[nginx.conf](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task6/nginx.conf)
