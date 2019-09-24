
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
2. Deploy to Development - deploy image from #1 to DEV env
3. Promote to Testing (approval) - asking user approval, if approved, tag image from #1 to promoteToQA, and deploy to TEST env
4. Deploy to Production (approval) - asking user approval, if approved, tag image from #3 to promoteToProd, and deploy to PROD env
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

4. Add edit role from userx-cicd:jenkins to userx-dev, userx-test and userx-prod
```
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-dev  
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-test    
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-prod  
```
5. Import pipeline template from https://raw.githubusercontent.com/adithaha/workshop-cicd/master/sample-php-website/pipeline-sample-php-website.yaml
```
oc create -f https://raw.githubusercontent.com/adithaha/workshop-cicd/master/sample-php-website/pipeline-sample-php-website.yaml
```
6. Configure jenkinsfile
```
oc edit bc/pipeline-sample-php-website
```
Replace jenkinsfile below:
```
        node() {
        stage 'build'
        openshiftBuild(buildConfig: 'myphp', showBuildLogs: 'true')
        }
```
with this:
```
        node('') {
        stage 'Build'
        openshiftBuild(namespace: 'userx-dev', buildConfig: 'sample-php-website', showBuildLogs: 'true')
        stage 'Deploy to Development'
        openshiftDeploy(namespace: 'userx-dev', deploymentConfig: 'sample-php-website')
        openshiftScale(namespace: 'userx-dev', deploymentConfig: 'sample-php-website',replicaCount: '1')
        stage 'Promote to Testing'
        input 'Promote to Testing?'
        openshiftTag(namespace: 'userx-dev', sourceStream: 'sample-php-website',  sourceTag: 'latest', destinationStream: 'sample-php-website', destinationTag: 'promoteToQA')
        openshiftDeploy(namespace: 'userx-test', deploymentConfig: 'sample-php-website', )
        openshiftScale(namespace: 'userx-test', deploymentConfig: 'sample-php-website',replicaCount: '1')
        stage 'Promote to Production'
        input 'Promote to Production?'
        openshiftTag(namespace: 'userx-dev', sourceStream: 'sample-php-website',  sourceTag: 'promoteToQA', destinationStream: 'sample-php-website', destinationTag: 'promoteToProd')
        openshiftDeploy(namespace: 'userx-prod', deploymentConfig: 'sample-php-website', )
        openshiftScale(namespace: 'userx-prod', deploymentConfig: 'sample-php-website',replicaCount: '1')
        }
```
7. Jenkins will take some time before it ready to process pipeline
```
oc get pod | grep build
```
Note pod name, tail build log
```
oc logs -f <pod-name> --tail=100
```
Wait until message "Push successful" before continue


### Walkthrough the configurations via web console

Now we have all environment and CI/CD pipeline set. To gain more understanding, we will take a look the configurations via web console

1. Open and login into OpenShift web console via browser
```
https://master.jakarta-e3ab.open.redhat.com
```
2. Go to DEV environment (userx-dev), check if application is up and running (blue circle)
3. Go to TEST environment (userx-test), check if application is up and running (blue circle)
4. Go to PROD environment (userx-prod), check if application is up and running (blue circle)
5. Go to CI/CD environment (userx-cicd), check if jenkins is up and running (blue circle)
6. Check if pipeline already set
```
Build - Pipelines - jenkinsfile (pipeline-sample-php-website)
```

### Try out the pipeline

1. Go to pipeline
```
Build - Pipelines - pipeline-sample-php-website
```
2. Start Pipeline
3. You should see some progress up to Promote to Testing
```
Build - Deploy to Development - Promote to Testing (Input Required)
```
4. Approve Promote to Testing
```
Click Input Required, you will be redirected to jenkins page
Login with OpenShift, fill openshift username/password
Give authorize access, Allow selected permissions
Promote to Testing? Proceed
```
5. Go back to openshift web, you will see Promote to Testing is progressing
```
Build - Deploy to Development - Promote to Testing - Promote to Production (Input Required)
```
6. Approve Promote to Production, procedure is similar to #4
7. Now pipeline process is completed, is you see some errors, recheck the step again in section Configure Pipeline

### Scenario 1: sample application is changed in DEV, but not approved in TEST

Here we will try to start pipeline with scenario above. Wait for instructor to change the source code.

Expected result:
1. DEV is updated
2. TEST is not updated
3. PROD is not updated

After instructor give the instruction, follow procedure below.

1. Go to pipeline
```
Build - Pipelines - pipeline-sample-php-website
```
2. Start Pipeline
3. You should see some progress up to Promote to Testing
```
Build - Deploy to Development - Promote to Testing (Input Required)
```
4. Reject Promote to Testing
```
Click Input Required, you will be redirected to jenkins page
Promote to Testing? Abort
```
5. Go back to openshift web, you will see Promote to Testing is aborted, pipeline process is finished
6. Go to each application URL in DEV/TEST/PROD, check if expected result is correct

### Scenario 2: sample application is changed in DEV, promoted to TEST, rejected to PROD

Here we will try to start pipeline with scenario above. Wait for instructor to change the source code.

Expected result:
1. DEV is updated
2. TEST is updated
3. PROD is not updated

After instructor give the instruction, follow procedure below.

1. Go to pipeline
```
Build - Pipelines - pipeline-sample-php-website
```
2. Start Pipeline
3. You should see some progress up to Promote to Testing
```
Build - Deploy to Development - Promote to Testing (Input Required)
```
4. Approve Promote to Testing
```
Click Input Required, you will be redirected to jenkins page
Promote to Testing? Proceed
```
5. Go back to openshift web, you will see Promote to Testing is progressing
```
Build - Deploy to Development - Promote to Testing - Promote to Production (Input Required)
```
6. Reject Promote to Production
7. Go back to openshift web, you will see Promote to Production is aborted, pipeline process is finished
8. Go to each application URL in DEV/TEST/PROD, check if expected result is correct

### Scenario 3: sample application is changed in DEV, promoted to TEST, and promoted to PROD

Here we will try to start pipeline with scenario above. Wait for instructor to change the source code.

Expected result:
1. DEV is updated
2. TEST is updated
3. PROD is updated

After instructor give the instruction, follow procedure below.

1. Go to pipeline
```
Build - Pipelines - pipeline-sample-php-website
```
2. Start Pipeline
3. You should see some progress up to Promote to Testing
```
Build - Deploy to Development - Promote to Testing (Input Required)
```
4. Approve Promote to Testing
```
Click Input Required, you will be redirected to jenkins page
Promote to Testing? Proceed
```
5. Go back to openshift web, you will see Promote to Testing is progressing
```
Build - Deploy to Development - Promote to Testing - Promote to Production (Input Required)
```
6. Approve Promote to Production
7. Go back to openshift web, you will see Promote to Production is progressing, wait until pipeline process is finished
8. Go to each application URL in DEV/TEST/PROD, check if expected result is correct



