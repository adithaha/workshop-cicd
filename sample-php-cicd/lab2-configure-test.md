
Note: For all procedure below, replace all userx to your user account, eg. user4.

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
