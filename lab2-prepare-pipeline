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
6. Jenkins will take some time before it ready to process pipeline
```
oc get pod | grep jenkins
```
There are multiple pod, pick one with name without "-deploy" and status running, note pod name , tail its log
```
oc logs -f <pod-name> --tail=100
```
Wait until message below before continue to next section
```
INFO: Jenkins is fully up and running
```

### Configure jenkinsfile via web console

Now we have all environment and Jenkins set. Now we will check the environments and configure jenkinsfile pipeline configurations via web console

1. Open and login into OpenShift web console via browser
```
https://master.jakarta-e3ab.open.redhat.com
```
2. Go to DEV environment (userx-dev), check if application is up and running (blue circle)
3. Go to TEST environment (userx-test), check if application is up and running (blue circle)
4. Go to PROD environment (userx-prod), check if application is up and running (blue circle)
5. Go to CI/CD environment (userx-cicd), check if jenkins is up and running (blue circle)
6. Configure pipeline
```
Build - Pipelines - pipeline-sample-php-website - Actions - Edit
```
Replace all its content to below:
```
def templateName = 'sample-php-website'

node {
    stage('Build and Deploy to DEV') {
        openshift.withCluster() {
            openshift.withProject("userx-dev") {
                echo "Using project: ${openshift.project()}"
	            openshift.selector('bc', templateName).startBuild("--wait=true")
            }
        }
    }
    stage('Deploy to DEV') {
        openshift.withCluster() {
            openshift.withProject("userx-dev") {
                openshift.selector('dc', templateName).rollout()
            }
        }
    }
    stage('Promote to TEST') {
        input message: "Promote to TEST ?"
        openshift.withCluster() {
            openshift.withProject("userx-dev") {
                echo "Using project: ${openshift.project()}"
	            openshift.tag("${templateName}:latest", "${templateName}:promoteToQA") 
            }
            openshift.withProject("userx-test") {
                echo "Using project: ${openshift.project()}"
	            openshift.selector('dc', templateName).rollout()
            }
        }
    }
    stage('Promote to PROD') {
        input message: "Promote to PROD ?"
        openshift.withCluster() {
            openshift.withProject("userx-dev") {
                echo "Using project: ${openshift.project()}"
	            openshift.tag("${templateName}:promoteToQA", "${templateName}:promoteToProd") 
            }
            openshift.withProject("userx-prod") {
                echo "Using project: ${openshift.project()}"
	            def dc = openshift.selector('dc', templateName).rollout()
            }
        }
    }
}
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
