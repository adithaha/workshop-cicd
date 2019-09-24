
https://github.com/adithaha/workshop-cicd.git


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
4. Create route to the application with port mapping 8080
```
oc expose service sample-php-website --name=sample-php-website --port=8080
```
5. Check if application is deployed correctly in dev environment
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

### Configure pipeline 

Now we have all deployment set in dev, test and prod. Note down the URL, for example:
```
DEV: http://sample-php-website-userx-dev.apps.rhpds311.openshift.opentlc.com
TEST: http://sample-php-website-userx-test.apps.rhpds311.openshift.opentlc.com
PROD: http://sample-php-website-userx-prod.apps.rhpds311.openshift.opentlc.com
```
Now we will create CI/CD pipeline with step below:
```
1. Build - build application image and push it to DEV env
2. Deploy to DEV - deploy image from #1 to DEV env
3. Deploy to TEST - tag image from #1 to promoteToQA, and deploy to TEST env
4. Deploy to PROD (approval) - asking user approval, if approved, tag image from #3 to promoteToProd, and deploy to PROD env
```

1. Create CI/CD project
```
oc new-project userx-cicd  
```
2. Deploy Jenkins application
```
oc new-app --template=jenkins-ephemeral
```
3. Set CPU and Memory resources for Jenkins 
```
oc edit dc/jenkins
```
Replace existing resource below:
```
        resources:
          limits:
            memory: 512Mi
```          
With this:       
```
        resources:
          limits:
            cpu: '2'
            memory: 2Gi
          requests:
            cpu: '1'
            memory: 1Gi      
```

3. Add edit role from userx-cicd:jenkins to userx-dev, userx-test and userx-prod
```
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-dev  
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-test    
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-prod  
```
4. import pipeline yaml from https://raw.githubusercontent.com/adithaha/workshop-cicd/master/sample-php-website/pipeline-sample-php-website.yaml
```
oc create -f https://raw.githubusercontent.com/adithaha/workshop-cicd/master/sample-php-website/pipeline-sample-php-website.yaml
```

(import pipeline-sample-php-website.yaml) - https://github.com/adithaha/workshop-cicd/raw/master/sample-php-website/pipeline-sample-php-website.yaml  
(modify jenkinsfile) - https://github.com/adithaha/workshop-cicd/raw/master/sample-php-website/jenkinsfile  

### webhook

(create secret - webhook secret)  
(edit pipeline yaml)  
      triggers:  
        - generic:  
            secret: pipeline  
          type: Generic  
          
          
