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


[Схема drawio](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task1/InsureTech_to_be.drawio)
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

Список проблем и рисков, которые связаны с планируемым ростом нагрузки:
- синхронные взаимодействия не дадут возможности эффективно масштабироваться с ростом нагрузки
- сервисы связаны, выход из строя одного сервиса в цепочке синхронных запросов приведет к отказу всей цепочки
- пока идет опрос `ins-product-aggregator`, `core-app` ждет результата, увеличение нагрузки приведет к большим задержкам
- нет кеширования данных или об этом не сказано
- данные актуализируются сервисами независимо с разной частотой и возможна проблема несогласованности данных
- нет единой системы логирования или мониторинга

Предлагается перевести на EDA и Event-Streaming взаимодействия `core-app`, `ins-comp-settlement` и `ins-product-aggregator`:
- добавить между ними брокер сообщений Kafka с топиками актуальных продуктов и оформленных страховок
- `ins-product-aggregator` публикует актуальные предложения
- `core-app`, `ins-comp-settlement` получают данные об актуальных предложениях
- `core-app` через Transactional Outbox публикует заявки на оформление страховок

Сервисы становятся менее связными, обеспечивается лучшая масштабируемость, производительность, отказоустойчивость и надежность.

Предлагается использовать паттерн Transactional Outbox. Он решает проблему потери данных при взаимодействии между сервисами, а также повышает устойчивость и надёжность системы в условиях высоких нагрузок.
Этот шаблон поможет решить проблему несогласованной работы с данными в `core-app` и `ins-comp-settlement`, обеспечит надежную доставку события оформления полиса независимо от доступности `core-app`.

[InsureTech-to-be.drawio](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task3/InsureTech-to-be.drawio):
![](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task3/InsureTech-to-be.png)

# Task 4

Реализация osago-aggregator:
- Будем использовать свою БД для хранения заявок, форм ОСАГО
- Сервис `osago-aggregator` работает с `core-app` через брокер сообщений Kafka
- `core-app` и `osago-aggregator` будут интегрированы посредством взаимодействия "публикация-подписка", т.к. ответ от страховой компании происходит не в режиме онлайн (до 60 секунд), а большое количество одновременных пользователей, возможно, потребует отдельного масштабирования сервиса обработки заявок на ОСАГО
- В API для веб-приложения в `core-app` добавится обработка заявок на осаго с отправкой формы
- Интеграции между веб-приложением и `core-app` будет выполнена через server push и SSE из-за простоты и отстствия необходимости в онлайн коммуникации
- Rate Limiting нужно применить на стороне сервера `core-app` ко всем внешним входящим запросам чтобы снизить нагрузку и со стороны веб-клиента в платежный сервис для безопасности клиентов
- Circuit Breaker нужно применить на исходящие запросы на получение данных к внешним сервисам чтобы не перегружать партнеров
- Retry нужно применить совместно с Timeout к исходящим запросам во все внешние сервисы

[InsureTech-to-be.drawio](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task4/InsureTech-to-be.drawio):
![](https://github.com/Boropwnz/architecture-insuretech/blob/sprint_6/Task4/InsureTech-to-be.png)

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
