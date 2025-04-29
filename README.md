# architecture-insuretech

# Task 1

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
