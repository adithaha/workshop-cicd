
Note: For all procedure below, replace all userx to your user account, eg. user4.

### Configure deployment on development environment

1. Login into openshift (you need openshift web console url correctly)
```
oc login -u userx https://<openshift-cli-console>
```
2. Create development project
```
oc new-project <userx>-dev
```
### Deploy Postgre DB in development environment

1. Deploy postgre
```
oc new-app postgresql-ephemeral -e POSTGRESQL_USER=postgres -e POSTGRESQL_PASSWORD=postgres -e POSTGRESQL_DATABASE=postgres
```
### Deploy nationalparks backend in development environment

1. Deploy backend application deployment using java image and source code from git
```
oc policy add-role-to-user view system:serviceaccount:<userx>-dev:default
oc create -f https://raw.githubusercontent.com/adithaha/nationalparks/master/ose3/application-template.json
oc new-app nationalparks-postgre
```
It may take 5-10 minutes to complete build the application, you can monitor in openshift web console, see if there sign '1 of 1 pods' on the right side of nationalparks application.

2. Check if backend application is deployed correctly in dev environment
```
oc get route
```
Note the nationalparks application host url, open it in a browser, add /ws/data/all path. You should be able to get api response.

ex.
```
http://nationalparks-anugraha-msa.apps.rhpds3x.openshift.opentlc.com/ws/data/all
```

### Deploy parksmap web in development environment

1. Deploy web application deployment using java image and source code from git
```
oc create -f https://raw.githubusercontent.com/adithaha/parksmap-web/master/ose3/application-template.json
oc new-app parksmap-web
```
It may take 5-10 minutes to complete build the application, you can monitor in openshift web console, see if there sign '1 of 1 pods' on the right side of nationalparks application.

2. Check if web application is deployed correctly in dev environment
```
oc get route
```
Note the parksmap-web application host url, open it in a browser. 

You should be able to see map with park geo location
ex.
```
http://parksmap-anugraha-msa.apps.rhpds3x.openshift.opentlc.com/
```
