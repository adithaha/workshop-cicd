### Deploy CI/CD Engine - SKIP THIS!! CONTINUE TO PREPARE PIPELINE ###

1. Create CI/CD project
```
oc new-project cicd  
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
