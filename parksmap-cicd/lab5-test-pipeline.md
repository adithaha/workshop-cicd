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



