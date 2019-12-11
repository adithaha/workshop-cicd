
Note: For all procedure below, replace all userx to your user account, eg. user4.

### Configure deployment on test environment

1. Tag previous image from development to promoteToQA
```
oc tag <userx>-dev/nationalparks:latest <userx>-dev/nationalparks:promoteToQA
oc tag <userx>-dev/parksmap:latest <userx>-dev/parksmap:promoteToQA  
```
2. Create test project
```
oc new-project <userx>-test  
```
3. Add image-puller role from userx-test to userx-dev, so test env can pull image from dev env
```
oc policy add-role-to-group system:image-puller system:serviceaccounts:<userx>-test -n <userx>-dev  
```
4. Add view service account to project <userx>-test
```
oc policy add-role-to-user view system:serviceaccount:<userx>-test:default
```
5. Deploy postgre
```
oc new-app postgresql-ephemeral -e POSTGRESQL_USER=postgres -e POSTGRESQL_PASSWORD=postgres -e POSTGRESQL_DATABASE=postgres
```

6. Deploy image with promoteToQA tag to test environment
```
oc new-app --image-stream=<userx>-dev/nationalparks:promoteToQA --name=nationalparks
oc new-app --image-stream=<userx>-dev/parksmap:promoteToQA --name=parksmap
```
7. Create route to the application with port mapping 8080
```
oc expose service nationalparks --name=nationalparks --port=8080
oc expose service parksmap --name=parksmap --port=8080
```
8. Check if application is deployed correctly in test environment
```
oc get route
```
Note the backend and web  url, open it in a browser. You should able to see response from both
