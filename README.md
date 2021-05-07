# Setup 

mkdir jenkins-data/ nexus-data/ sonarqube/

docker-compose up -d

docker ps devrait afficher 
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
e07b354b0302   jenkinsci/blueocean   "/sbin/tini -- /usr/…"   3 seconds ago   Up 2 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 50000/tcp   jenkins
a85779621db3   sonarqube             "bin/run.sh bin/sona…"   4 seconds ago   Up 3 seconds   0.0.0.0:9000->9000/tcp, :::9000->9000/tcp              sonarqube
7a6a21b6f89c   sonatype/nexus3       "sh -c ${SONATYPE_DI…"   4 seconds ago   Up 3 seconds   0.0.0.0:8081->8081/tcp, :::8081->8081/tcp              nexus


docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
Rentrer le mot de passe dans localhost:8080
Installer les plugins suggérés
Créer un administrateur admin admin
Laisser l'URL d'instance par défaut (localhost:8080)
Start using Jenkins
Manage Jenkins -> Manage Plugin -> Available -> SonarQube Scanner -> install without restart
docker restart jenkins
Actualiser la page, se loger avec le compte admin créer plus tôt


SI CA FONCTIONNE ?
ifconfig ou ipconfig
en0 ou enp0
Copier son addresse ip
ADRESSE IP UNIQUE POUR LES 2 CONFIGURATIONS

SI CA FONCTIONNE PAS
get jenkins container ip
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' jenkins 
get sonarqube container ip
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' sonarqube


localhost:9000 admin admin
Changer de mot de passe
Administration -> Configuration -> webhook
Creer un nouveau webhook 
name : Sonarqube
URL : http://<my-jenkins-ip>:8080/sonarqube-webhook
Create
Administrator -> my account -> security 
jenkins puis generate token
Copier le token créé
761d886664b08d3a93057a4ce76fca96b4a73254


Manage Jenkins -> Configure Global Tool -> Sonarqube Scanner
Name : Sonarqube scanner
Check install automatically
Save

Manage Jenkins -> Configure System -> SonarQube servers
Name : Sonarqube
server URL : http://<my-sonarqube-ip>:9000
Server authentication token -> add 
Kind : Secret text
Secret : le token copier plus haut
id: token sonarqube
Save


Sonarqube -> create new project
project key : projet
OK
Provide token -> use existing token -> le nom du token créé plus tôt
OK
Other -> Linux


Jenkins
New Item -> Freestyle project 
Source Code Management -> Git 
URL : URL du depôt git
Branch : */main
Build -> add build step -> Sonarqube Scanner
Analysis properties : sonar.projectKey=projet
OK

# Tweeks
Pour build après chaque commit
Jenkins -> Manage Jenkins -> Manage Plugin -> Github Integration -> install without restart
Restart jenkins
Dans le projet github -> settings -> webhooks -> add new webhook
Payload URL : http://<my-jenkins-ip>:8080/github-webhook
Content type : json
Check Just push event
Check Active

Alternative 

Jenkins -> projet -> configure -> build triggers -> Check poll SCM et mettre * * * * * pour check toutes les minutes
