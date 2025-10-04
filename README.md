# Dépôt des sources JAVA pour cours CICD 

Sources accompagnant le cours "Continuous Integration, Continuous Delivery"

def runApp(containerName, tag, dockerHubUser, httpPort, envName) {
    sh "docker pull $dockerHubUser/$containerName:$tag"
    sh "docker run --network springmysql-net --env SPRING_ACTIVE_PROFILES=$envName -d -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
 // sh "docker run --rm --env SPRING_ACTIVE_PROFILES=$envName -d -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}

--network springmysql-net : je doit creer le reseau si dans la machine virtuelle si non sa ne va pas fonctioner 

et je dois aussi 
  url: jdbc:mysql://mysqldb:3306/calculator?serverTimezone=UTC
```docker run --name mysqldb --network springmysql-net -e MYSQL_ROOT_PASSWORD=manounou -e MYSQL_DATABASE=calculator -e MYSQL_USER=stevy -e MYSQL_PASSWORD=manounou -d mysql:5.7```

SPRING_ACTIVE_PROFILES=$envName : si env = prod sa va lire application-prod.yaml si env = uat sa va lire application-uat.yaml si env = dev sa va lire application-dev.yaml
