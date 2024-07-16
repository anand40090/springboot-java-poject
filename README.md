# CI-CD end to end pieline using maven, docker
--------------
# Jenkins CI-CD pipeline using docker, maven

________________

In this project we are building CI-CD pipeline for java based project using maven and docker tool. 

_______________

## Seting up tools 

1. Install the required plugins and tools in the jenkins
   - Docker
   - Sonarqube
   - Maven
   - JDK
2. Configure the tools in Jenkins
3. Create tocken for Sonarqube and configure it's in jenkins 
4. Configure docker credentials in jenkins

_________________________

# Configure the tools, credentials in jenkins 

1. configure credentials for sonqrqube and docker 
```
Jenkins Dashboard >> Manage Jenkins >> Credentials >> System >> Global credentials (unrestricted)

```

![image](https://github.com/user-attachments/assets/c5bc6d3e-26e6-4a20-bf21-88f6f610342e)

2. configure the maven in jenkins 

![image](https://github.com/user-attachments/assets/bbc01a38-a843-4175-b040-cd848f4d449f)

3. configure sonarqube in jenkins 

![image](https://github.com/user-attachments/assets/e61a8afc-d57a-4acc-8c7a-e41b9947a7bd)

# Create webhook in sonarqube for jenkins
1. Login to sonarqube and created PAT token to be configured in the jenkins, in above screenshot this token have be saved in the jenkins credentials 'sq-1'

![image](https://github.com/user-attachments/assets/bdfaaf3f-a646-4eab-a99b-cc4372c2b171)

2. configure webhook using jenkins URL 

![image](https://github.com/user-attachments/assets/72f96fc5-774e-4c22-b60b-a8e76742f045)
