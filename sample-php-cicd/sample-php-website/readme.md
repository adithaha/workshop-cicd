# Simple PHP Website

## Deploy on OpenShift with Pipeline

### deploy in development

oc new-project development  
(create ssh secret with name redhat-id via gui)  
oc new-app --image-stream=openshift/php:5.6 --code=https://gitlab.com/redhat-id/kai/sample-php-website.git --name=sample-php-website --source-secret=redhat-id  
oc expose svc sample-php-website  

### deploy in testing

oc tag development/sample-php-website:latest development/sample-php-website:promoteToQA  
oc new-project testing  
oc policy add-role-to-group system:image-puller system:serviceaccounts:testing -n development  
oc new-app --image-stream=development/sample-php-website:promoteToQA name=sample-php-website  
oc expose svc sample-php-website  

### deploy in production

oc tag development/sample-php-website:promoteToQA development/sample-php-website:promoteToProd  
oc new-project production  
oc policy add-role-to-group system:image-puller system:serviceaccounts:production -n development  
oc new-app --image-stream=development/sample-php-website:promoteToProd name=sample-php-website  
oc expose svc sample-php-website  


### using pipeline 

oc new-project cicd  
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n development  
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n testing  
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n production  
(import pipeline-sample-php-website.yaml)  
(modify jenkinsfile)  

### webhook

(create secret - webhook secret)  
(edit pipeline yaml)  
      triggers:  
        - generic:  
            secret: pipeline  
          type: Generic  
          
          
curl -k -X POST https://kai.openshift.cluster.up:8443/oapi/v1/namespaces/cicd/buildconfigs/pipeline-sample-php-website/webhooks/pipeline/generic  
