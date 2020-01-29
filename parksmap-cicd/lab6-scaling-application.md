## Scaling your application

OpenShift has capability to scale your application vertically (increasing resource) and horizontally (increasing app instances). Both are very flexible to support monolith and microservices application deployment.

## Vertical Scaling

Here we will increase resource for parksmap web application via web console
1. Go to <userx>-prod project
```
Application - Deployment - parksmap - Action - Edit Resource Limit
```
2. Increase resource as below, replace resources: {}
```
          resources:
            limits:
              cpu: 500m
              memory: 512Mi
            requests:
              cpu: 100m
              memory: 256Mi
```
3. Your application will be redeployed with configuration above

## Horizontal Scaling

Here we will increase instance of nationalpark backend application via web console
1. Go to <userx>-prod project
2. Home - Status - parksmap - desired count
2. Increase instance number from 1 to 2 using + icon, click Save
3. New identical instance will be deployed, with same route, and automatically load balanced. It will be seamless from client perspective
