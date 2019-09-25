
Note: For all procedure below, replace all userx to your user account, eg. user4.

### Configure deployment on development environment

1. Login into openshift (you need openshift web url correctly)
```
oc login -u userx https://master.jakarta-e3ab.open.redhat.com
```
2. Create development project
```
oc new-project userx-dev
```
3. Create new application deployment using PHP image and source code from git
```
oc new-app php:7.1~https://github.com/adithaha/workshop-cicd.git --context-dir=/sample-php-website --name=sample-php-website
```
4. Application is being build now, check build log
```
oc get pod | grep build
```
Note pod name, tail build log
```
oc logs -f <pod-name> --tail=100
```
Wait until message "Push successful" before continue

5. Create route to the application with port mapping 8080
```
oc expose service sample-php-website --name=sample-php-website --port=8080
```
6. Check if application is deployed correctly in dev environment
```
oc get route
```
Note the application host url, open it in a browser. You should able to see sample PHP application web page (dev)


### Configure deployment on test environment

1. Tag previous image from development to promoteToQA
```
oc tag userx-dev/sample-php-website:latest userx-dev/sample-php-website:promoteToQA  
```
2. Create test project
```
oc new-project userx-test  
```
3. Add image-puller role from userx-test to userx-dev, so test env can pull image from dev env
```
oc policy add-role-to-group system:image-puller system:serviceaccounts:userx-test -n userx-dev  
```
4. Deploy image with promoteToQA tag to test environment
```
oc new-app --image-stream=userx-dev/sample-php-website:promoteToQA --name=sample-php-website
```
5. Create route to the application with port mapping 8080
```
oc expose service sample-php-website --name=sample-php-website --port=8080
```
6. Check if application is deployed correctly in test environment
```
oc get route
```
Note the application host url, open it in a browser. You should able to see sample PHP application web page (test)

### Configure deployment on prod environment

1. Tag previous image from development to promoteToProd
```
oc tag userx-dev/sample-php-website:promoteToQA userx-dev/sample-php-website:promoteToProd   
```
2. Create prod project
```
oc new-project userx-prod  
```
3. Add image-puller role from userx-prod to userx-dev, so prod env can pull image from dev env
```
oc policy add-role-to-group system:image-puller system:serviceaccounts:userx-prod -n userx-dev  
```
4. Deploy image with promoteToProd tag to prod environment
```
oc new-app --image-stream=userx-dev/sample-php-website:promoteToProd --name=sample-php-website
```
5. Create route to the application with port mapping 8080
```
oc expose service sample-php-website --name=sample-php-website --port=8080
```
6. Check if application is deployed correctly in prod environment
```
oc get route
```
Note the application host url, open it in a browser. You should able to see sample PHP application web page (prod)