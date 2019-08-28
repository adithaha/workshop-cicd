
https://github.com/adithaha/workshop-cicd.git


# Simple PHP Website

## Deploy on OpenShift with Pipeline

### deploy in development

oc new-project development  
(create ssh secret with name redhat-id via gui)  
oc new-app --image-stream=openshift/php:5.6 --code=https://gitlab.com/redhat-id/kai/sample-php-website.git --name=sample-php-website --source-secret=redhat-id  
oc expose svc sample-php-website  

### deploy in testing

oc tag userx-dev/sample-php-website:latest userx-dev/sample-php-website:promoteToQA  
oc policy add-role-to-group system:image-puller system:serviceaccounts:userx-test -n userx-dev  

### deploy in production

oc tag userx-dev/sample-php-website:promoteToQA userx-dev/sample-php-website:promoteToProd  
oc policy add-role-to-group system:image-puller system:serviceaccounts:userx-prod -n userx-dev  



### using pipeline 

oc new-project cicd  
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n userx-dev  
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n userx-test    
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n userx-prod  
(import pipeline-sample-php-website.yaml)  
(modify jenkinsfile)  

### webhook

(create secret - webhook secret)  
(edit pipeline yaml)  
      triggers:  
        - generic:  
            secret: pipeline  
          type: Generic  
          
          
