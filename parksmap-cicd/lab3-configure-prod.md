
Note: For all procedure below, replace all userx to your user account, eg. user4.

### Configure deployment on prod environment

1. Tag previous image from promoteToQA to promoteToProd
```
oc tag <userx>-dev/nationalparks:promoteToQA <userx>-dev/nationalparks:promoteToProd
oc tag <userx>-dev/parksmap:promoteToQA <userx>-dev/parksmap:promoteToProd  
```
2. Create test project
```
oc new-project <userx>-prod  
```
3. Add image-puller role from userx-prod to userx-dev, so prod env can pull image from dev env
```
oc policy add-role-to-group system:image-puller system:serviceaccounts:<userx>-prod -n <userx>-dev  
```
4. Add view service account to project <userx>-prod
```
oc policy add-role-to-user view system:serviceaccount:<userx>-prod:default
```
5. Deploy postgre
```
oc new-app postgresql-ephemeral -e POSTGRESQL_USER=postgres -e POSTGRESQL_PASSWORD=postgres -e POSTGRESQL_DATABASE=postgres
```

6. Deploy image with promoteToProd tag to test environment
```
oc new-app --image-stream=<userx>-dev/nationalparks:promoteToProd --name=nationalparks -e POSTGRE_HOST=postgresql -e POSTGRE_PORT=5432 -e POSTGRE_DB_NAME=postgres -e POSTGRE_USERNAME=postgres -e POSTGRE_PASSWORD=postgres
oc new-app --image-stream=<userx>-dev/parksmap:promoteToProd --name=parksmap
```
7. Create route to the application with port mapping 8080
```
oc expose service nationalparks --name=nationalparks --port=8080
oc expose service parksmap --name=parksmap --port=8080
```
8. Check if application is deployed correctly in prod environment
```
oc get route
```
Note the backend and web  url, open it in a browser. You should able to see response from both
