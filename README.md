## Jenkins with OpenJDK 11
### into Alpine OS with OpenJDK 11
### Maven for buildings
### Docker in Docker
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

#### Build
```
docker build -t izone/jenkins .
```

