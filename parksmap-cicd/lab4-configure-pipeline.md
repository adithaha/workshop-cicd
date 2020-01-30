### Configure pipeline 

Now we have all deployment set in dev, test and prod. Note down the web URL, for example:
```
DEV: http://parksmap-user99-dev.apps.cluster-jakarta-fbff.jakarta-fbff.example.opentlc.com
TEST: http://parksmap-user99-dev.apps.cluster-jakarta-fbff.jakarta-fbff.example.opentlc.com
PROD: http://parksmap-user99-dev.apps.cluster-jakarta-fbff.jakarta-fbff.example.opentlc.com
```
Now we will create CI/CD pipeline for web component with step below:
```
1. Build - build application image and push it to DEV env
2. Deploy to Development - deploy image from #1 to DEV env
3. Promote to Testing (approval) - asking user approval, if approved, tag image from #1 to promoteToQA, and deploy to TEST env
4. Deploy to Production (approval) - asking user approval, if approved, tag image from #3 to promoteToProd, and deploy to PROD env
```
### Prepare Pipeline ###

1. Go to  CI/CD project
```
oc project cicd  
```
2. Add edit role from userx-cicd:jenkins to userx-dev, userx-test and userx-prod
```
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n <userx>-dev  
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n <userx>-test    
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n <userx>-prod  
```
### Configure jenkinsfile via web console

Now we have all environment and Jenkins set. Now we will check the environments and configure jenkinsfile pipeline configurations via web console

1. Open and login into OpenShift web console via browser
```
https://<openshift>
```
2. Go to DEV environment (userx-dev), check if application is up and running (blue circle)
3. Go to TEST environment (userx-test), check if application is up and running (blue circle)
4. Go to PROD environment (userx-prod), check if application is up and running (blue circle)
5. Go to CI/CD environment (userx-cicd), check if jenkins is up and running (blue circle)
6. Create pipeline in CI/CD environment
```
Add to Project - Import YAML - paste pipeline template script below - change name to your specific user
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
Builds - Build Configs - <userx>-parksmap-pipeline - Edit Build Config
```
Replace jenkinsfile script under 'jenkinsfile: |-' tag with below. Don't forget to put correct <userx>:
```
        def templateName = 'parksmap'

        node {
            stage('Build and Deploy to DEV') {
                openshift.withCluster() {
                    openshift.withProject("<userx>-dev") {
                        echo "Using project: ${openshift.project()}"
                          openshift.selector('bc', templateName).startBuild("--wait=true")
                    }
                }        
            }
            stage('Deploy to DEV') {
                openshift.withCluster() {
                    openshift.withProject("<userx>-dev") {
                        openshift.selector('dc', templateName).rollout()
                    }
                }
            }
            stage('Promote to TEST') {
                input message: "Promote to TEST ?"
                openshift.withCluster() {
                    openshift.withProject("<userx>-dev") {
                        echo "Using project: ${openshift.project()}"
                          openshift.tag("${templateName}:latest", "${templateName}:promoteToQA") 
                    }
                    openshift.withProject("<userx>-test") {
                        echo "Using project: ${openshift.project()}"
                          openshift.selector('dc', templateName).rollout()
                    }
                }
            }
            stage('Promote to PROD') {
                input message: "Promote to PROD ?"
                openshift.withCluster() {
                    openshift.withProject("<userx>-dev") {
                      echo "Using project: ${openshift.project()}"
                        openshift.tag("${templateName}:promoteToQA", "${templateName}:promoteToProd") 
                    }
                    openshift.withProject("<userx>-prod") {
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
Builds - Build Config - <userx>-parksmap-pipeline
```
2. Start Build
3. You should see some progress up to Promote to Testing
```
Build - Deploy to Development - Promote to Testing (Input Required)
```
4. Approve Promote to Testing
```
Click Input Required, you will be redirected to jenkins page
Login with OpenShift, fill openshift username/password
Give authorize access, Allow selected permissions
In left panel, select Paused for Input
Promote to Testing? Proceed
```
5. Go back to openshift web, you will see Promote to Testing is progressing
```
Build - Deploy to Development - Promote to Testing - Promote to Production (Input Required)
```
6. Approve Promote to Production, procedure is similar to #4
7. Now pipeline process is completed, is you see some errors, recheck the step again in section Configure Pipeline
