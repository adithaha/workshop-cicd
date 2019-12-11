### Configure pipeline 

Now we have all deployment set in dev, test and prod. Note down the web URL, for example:
```
DEV: http://parksmap-userx-dev.apps.rhpds3x.openshift.opentlc.com
TEST: http://parksmap-userx-dev.apps.rhpds3x.openshift.opentlc.com
PROD: http://parksmap-userx-dev.apps.rhpds3x.openshift.opentlc.com
```
Now we will create CI/CD pipeline for web component with step below:
```
1. Build - build application image and push it to DEV env
2. Deploy to Development - deploy image from #1 to DEV env
3. Promote to Testing (approval) - asking user approval, if approved, tag image from #1 to promoteToQA, and deploy to TEST env
4. Deploy to Production (approval) - asking user approval, if approved, tag image from #3 to promoteToProd, and deploy to PROD env
```

### Deploy CI/CD Engine - skip if instructor has created this for you ###

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
            memory: 1536Mi
          requests:
            cpu: '1'
            memory: 1024Mi      
```
```
oc adm policy add-role-to-user admin user1 -n cicd
oc adm policy add-role-to-user admin user2 -n cicd
oc adm policy add-role-to-user admin user3 -n cicd
oc adm policy add-role-to-user admin user4 -n cicd
oc adm policy add-role-to-user admin user5 -n cicd
oc adm policy add-role-to-user admin user6 -n cicd
oc adm policy add-role-to-user admin user7 -n cicd
oc adm policy add-role-to-user admin user8 -n cicd
oc adm policy add-role-to-user admin user9 -n cicd
oc adm policy add-role-to-user admin user10 -n cicd
oc adm policy add-role-to-user admin user11 -n cicd
oc adm policy add-role-to-user admin user12 -n cicd
oc adm policy add-role-to-user admin user13 -n cicd
oc adm policy add-role-to-user admin user14 -n cicd
oc adm policy add-role-to-user admin user15 -n cicd
oc adm policy add-role-to-user admin user16 -n cicd
oc adm policy add-role-to-user admin user17 -n cicd
oc adm policy add-role-to-user admin user18 -n cicd
oc adm policy add-role-to-user admin user19 -n cicd
oc adm policy add-role-to-user admin user20 -n cicd
oc adm policy add-role-to-user admin user99 -n cicd
```
### Prepare Pipeline ###

1. Add edit role from userx-cicd:jenkins to userx-dev, userx-test and userx-prod
```
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-dev  
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-test    
oc policy add-role-to-user edit system:serviceaccount:userx-cicd:jenkins -n userx-prod  
```
2. Import pipeline template from https://raw.githubusercontent.com/adithaha/workshop-cicd/master/sample-php-website/pipeline-sample-php-website.yaml
```
oc create -f https://raw.githubusercontent.com/adithaha/workshop-cicd/master/sample-php-website/pipeline-sample-php-website.yaml
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
6. Create pipeline in CI/CD environment
```
Add to Project - Import YAML/JSON - paste below - change name to your specific user
```
```
apiVersion: v1
kind: BuildConfig
metadata:
  name: <userx>-parksmap-pipeline
  labels:
    name: <userx>-parksmap-pipeline
  annotations:
    pipeline.alpha.openshift.io/uses: '[{"name": "<userx>-parksmap-pipeline", "namespace": "", "kind": "DeploymentConfig"}]'
spec:
  runPolicy: Serial
  source:
    type: None
  strategy:
    type: JenkinsPipeline
    jenkinsPipelineStrategy:
      jenkinsfile: "node() {\nstage 'build'\nopenshiftBuild(buildConfig: 'myphp', showBuildLogs: 'true')\n}"
  output:
  resources:
  postCommit:
```
7. Configure pipeline
```
Build - Pipelines - <userx>-parksmap-pipeline - Actions - Edit
```
Replace all its content to below:
```
def templateName = 'parksmap'

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
Build - Pipelines - <userx>-parksmap-pipeline
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
