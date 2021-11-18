## Jenkins 2.321 into Alpine OS with OpenJDK 11
### Maven, docker-compose and kubectl for buildings
### jenkins-agent 4.11
### Files to deploy on k8s 
-----

#### Pull image
```
docker pull izone/jenkins
```
#### Run
```
docker run --rm --name Jenkins \
--net=host \
-v /var/run/docker.sock:/var/run/docker.sock \
--mount type=bind,src=$HOME/jenkins,dst=/var/jenkins_home \
-d izone/jenkins
```
#### Access Browser
```
http://localhost:8080/
```

#### With docker-compose
```
docker-compose up -d jenkins
```
### (includes sonar if unspecified service)
```
docker-compose up
```
-----
#### Build
```
docker build -t izone/jenkins .
```
### Master
```
docker build -t izone/jenkins:master ./master
```
### Agent
```
docker build -t izone/jenkins:agent ./agent
```
