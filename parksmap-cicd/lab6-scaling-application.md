## Scaling your application

OpenShift has capability to scale your application vertically (increasing resource) and horizontally (increasing app instances). Both are very flexible to support monolith and microservices application deployment.

## Vertical Scaling

Here we will increase resource for parksmap web application via web console
1. Go to <userx>-prod project
```
Application - Deployment - parksmap - Action - Edit Resource Limit
```
2. Increase resource as below
```
CPU Request: 100 millicores
CPU Limit: 500 millicores
Memory Request: 256 MiB
Memory Request: 512 MiB
Save
```
3. Your application will be redeployed with configuration above

## Horizontal Scaling

Here we will increase instance of nationalpark backend application via web console
1. Go to <userx>-prod project
```
Overview
```
2. Increase instance number from 1 to 2 using up arrow
3. New identical instance will be deployed, with same route, and automatically load balanced. It will be seamless from client perspective
