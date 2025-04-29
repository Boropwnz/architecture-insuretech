# architecture-insuretech

# Task 1

Масштабирование нужно делать геораспределенным с CDS и GSLB, в ЦОД-ах расположенных в разных регионах и в разных частях регионов.
  
Стратегия отказоустойчивости должна быть active-active, т.к:
- отлично подходит для высоконагруженных систем с высокими требованиями к доступности и отказоустойчивости
- улучшает отклик приложения благодаря равномерному распределению нагрузки
- потенциальное время простоя стремится к нулю
- сбой одного узла приводит только к частичной недоступности
- хорошо сочетается с георезервированием

Для атоматического масштабирования кластеров kubernetes нужно использовать HPA+CA, т.к. это поможет:
- поддерживать нужный уровень производительности даже при внезапных всплесках трафика, за счёт поддержания количества подов, соизмеримого с текущей нагрузкой
- оптимизировать расходы, автоматически уменьшая количество подов в периоды низкой нагрузки

Сами кластеры kubernetes должны быть независимымы для повышения отказоустойчивости, большей простоты настройки, более удобного обслуживания и обновления, более оптимальной передачи и использования данных.
GSLB позволит делать health check и оптимально перенапрвлять запросы к ЦОДам. Обеспечит оптимальную скорость доступа и переключение на другой ЦОД в случае отказа.
Шаблон конфигураций Patroni будет отслеживать работоспособность позволит добиться высокой доступности и отказоустойчивости распределённого кластера PostgreSQL. Он будет использовать HAProxy для балансировки нагрузки.
Каждый экземпляр PostgreSQL в кластере поддерживает согласованность с другими узлами посредством потоковой репликации. Асинхронная репликация между ЦОДами не реже чем раз в 15 минут позволят достичь требуемого RPO.


[Схема drawio](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task1/InsureTech_%D1%82%D0%B5%D1%85%D0%BD%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F_%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0-to-be.drawio)

![Схема drawio](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task1/InsureTech_to_be.png)

# Task 2
Использовались комманды:
```
minikube start --vm-driver=docker --addons=metrics-server
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
minikube service scaletestapp-service --url
minikube dashboard
locust
```
Содержимое [deployment.yaml](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/deployment.yaml):
```
apiVersion: apps/v1
kind: Deployment                                         
metadata:
  name: scaletestapp-deployment
spec:
  replicas: 1         
  selector:
    matchLabels:
      app: scaletestapp
  template:                                               
    metadata:
      labels:
        app: scaletestapp
    spec:                                                
      containers:
        - image: shestera/scaletestapp
          name: scaletestapp                            
          ports:
            - containerPort: 80
          resources:
            limits:
              memory: "20Mi"
```

Содержимое [service.yaml](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/service.yaml):
```
apiVersion: v1
kind: Service
metadata:
  name: scaletestapp-service
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: scaletestapp
```

Содержимое [hpa.yaml](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/hpa.yaml):
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: scaletestapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: scaletestapp-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 60
```

До нагрузки:
![До нагрузки](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/before_load.jpg)

Нагрузка:
![Нагрузка](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/locust.jpg)

После нагрузки:
![осле нагрузки](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task2/after_load.jpg)

# Task 3

# Task 4

# Task 5
Содержимое [client-inf.graphql](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task5/client-inf.graphql):
```
type Query {
  clientInfo(clientId: ID!): Client
  clientDocuments(clientId: ID!): [Document]
  clientRelatives(clientId: ID!): [Relative]
}

type Client {
    id: ID!
    name: String!
    age: Int!
    documents: [Document]
    relatives: [Relative]
}

type Document {
    id: ID!
    type: String!
    number: String!
    issueDate: String!
    expiryDate: String!
}

type Relative {
  id: ID!
  relationType: String!
  name: String!
  age: Int
}
```

# Task 6
Содержимое [nginx.conf](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task6/nginx.conf):
```
http {
   upstream backend_servers {
       server backend1.example.com;
       server backend2.example.com;
       server backend3.example.com;
   }
   limit_req_zone $binary_remote_addr zone=one:10m rate=10r/m;

   server {
       listen 80;
       location / {
           limit_req zone=one burst=10 nodelay;
           limit_req_status 429;
           proxy_pass http://backend_servers;
       }
   }
}
```
