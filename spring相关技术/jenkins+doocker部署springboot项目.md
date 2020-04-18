```java
docker run \
  -u root \
  -d \
  -p 9001:8080 \
  -p 9002:50000 \
  -v /data/jenkins:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
```

```java
docker run -itd -p 9003:8080 -p 9004:50000  --restart always -v /apps/devops/jenkins:/var/jenkins_home --name jenkins  jenkins/jenkins:lts

```

