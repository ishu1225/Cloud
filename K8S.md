Deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-deploy
  labels:
    app: devops-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-app
  template:
    metadata:
      labels:
        app: devops-app
    spec:
      containers:
        - name: devops-app
          image: zen1tsu/devops-app:latest
          ports:
            - containerPort: 5000


service.yml

apiVersion: v1
kind: Service
metadata:
  name: devops-service
spec:
  type: NodePort
  selector:
    app: devops-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30082