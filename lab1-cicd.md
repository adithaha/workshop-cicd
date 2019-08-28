
https://github.com/adithaha/workshop-cicd.git



### login

oc login -u userx https://master.jakarta-e3ab.open.redhat.com

oc tag userx-dev/sample-php-website:latest userx-dev/sample-php-website:promoteToQA  
oc policy add-role-to-group system:image-puller system:serviceaccounts:userx-test -n userx-dev  

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
          
          
