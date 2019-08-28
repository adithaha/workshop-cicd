
https://github.com/adithaha/workshop-cicd.git



### login

oc login -u userx https://master.jakarta-e3ab.open.redhat.com

oc project userx-dev  

### Deploy to test environment

oc tag userx-dev/sample-php-website:latest userx-dev/sample-php-website:promoteToQA  

buat project userx-test

oc policy add-role-to-group system:image-puller system:serviceaccounts:userx-test -n userx-dev  

deploy image sample-php-website:promoteToQA  ke userx-test

### Deploy to prod environment

oc tag userx-dev/sample-php-website:promoteToQA userx-dev/sample-php-website:promoteToProd  

buat project userx-prod  
oc policy add-role-to-group system:image-puller system:serviceaccounts:userx-prod -n userx-dev  

deploy image sample-php-website:promoteToProd  ke userx-prod

### using pipeline 

oc new-project userx-cicd  
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-dev  
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-test    
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-prod  
(import pipeline-sample-php-website.yaml)  
(modify jenkinsfile)  

### webhook

(create secret - webhook secret)  
(edit pipeline yaml)  
      triggers:  
        - generic:  
            secret: pipeline  
          type: Generic  
          
          
