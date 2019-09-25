
Note: For all procedure below, replace all userx to your user account, eg. user4.

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
