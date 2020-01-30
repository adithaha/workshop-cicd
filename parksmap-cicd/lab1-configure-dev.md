
Note: For all procedure below, replace all userx to your user account, eg. user4.

### Configure deployment on development environment

1. Login into openshift (you need openshift web console url correctly)
```
oc login -u userx https://<openshift-web-console>
```
2. Create development project
```
oc new-project <userx-dev>
```
3. Deploy postgre
```
oc new-app postgresql-ephemeral -e POSTGRESQL_USER=postgres -e POSTGRESQL_PASSWORD=postgres -e POSTGRESQL_DATABASE=postgres
```

4. Deploy backend application deployment using java image and source code from git
```
oc policy add-role-to-user view system:serviceaccount:<userx-dev>:default
oc create -f https://raw.githubusercontent.com/adithaha/nationalparks/master/ose3/application-template.json
oc new-app nationalparks-postgre
```
5. Check if backend application is deployed correctly in dev environment
```
oc get route
```
Note the nationalparks application host url, open it in a browser, add /ws/data/all path. You should be able to get api response.

ex.
```
http://nationalparks-anugraha-msa.apps.rhpds3x.openshift.opentlc.com/ws/data/all
```
6. Deploy web application deployment using java image and source code from git
```
oc create -f https://raw.githubusercontent.com/adithaha/parksmap-web/master/ose3/application-template.json
oc new-app parksmap-web
```
7. Check if web application is deployed correctly in dev environment
```
oc get route
```
Note the parksmap-web application host url, open it in a browser. You should be able to see map with park geo location
ex.
```
http://parksmap-anugraha-msa.apps.rhpds3x.openshift.opentlc.com/
```
