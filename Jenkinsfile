def ENV_NAME = getEnvName(env.BRANCH_NAME)
def CONTAINER_NAME = "calculator-" +ENV_NAME
def CONTAINER_TAG = getTag(env.BUILD_NUMBER, env.BRANCH_NAME)
def HTTP_PORT = getHTTPPort(env.BRANCH_NAME)
def EMAIL_RECIPIENTS = "soniel1693@gmail.com" 


node {
    try {
        stage('Initialize') {
            def dockerHome = tool 'dockerlatest'
            def mavenHome = tool 'mavenlatest'
            def javaHome = tool 'Java 11'  // Force l'utilisation de Java 11
           
            env.JAVA_HOME = javaHome
            env.PATH = "${javaHome}/bin:${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
        }

        stage('Checkout') {
            checkout scm
        }

         stage('Verify Java Version') {
            sh 'java -version'
            sh 'echo "JAVA_HOME: $JAVA_HOME"'
            sh 'which java'
        }

        stage('Build with test') {
            //sh "mvn -X clean compile 2>&1 | grep -i compiler"
            
           
            sh "mvn clean compile"

              sh '''
                        echo "Java version par défaut:"
                        java -version
                        echo ""
                    '''
        }

  stage('Code Linting') {
            echo "🔍 Vérification de la qualité du code avec Checkstyle..."
            
            def checkstyleStatus = sh(
                script: 'mvn checkstyle:check',
                returnStatus: true
            )
            
            sh 'mvn checkstyle:checkstyle || true'
            
            archiveArtifacts artifacts: 'target/site/checkstyle.html', allowEmptyArchive: true
            archiveArtifacts artifacts: 'target/checkstyle-result.xml', allowEmptyArchive: true
            
            def violations = sh(
                script: "grep -oP '\\d+(?= errors reported)' target/checkstyle-result.xml 2>/dev/null || echo 0",
                returnStdout: true
            ).trim()
            
            echo "📊 Violations détectées: ${violations}"
            
            def checkstyleReport = """
                Violations: ${violations}
                Rapport HTML: ${env.BUILD_URL}artifact/target/site/checkstyle.html
                Rapport XML: ${env.BUILD_URL}artifact/target/checkstyle-result.xml
            """
            
            if (env.CHANGE_ID && checkstyleStatus != 0) {
                echo "❌ PULL REQUEST BLOQUÉE"
                echo "📋 Rapport disponible: ${env.BUILD_URL}artifact/target/site/checkstyle.html"
                
                sendEmail('soniel1693@gmail.com', checkstyleReport)
                
                error("Pull Request bloquée: ${violations} violations Checkstyle")
            }
            
            echo "ℹ️ Pipeline continue - Rapport Checkstyle disponible dans les artifacts"
            sendEmail('soniel1693@gmail.com', checkstyleReport)
        }
        stage('Build test') {
            //sh "mvn -X clean compile 2>&1 | grep -i compiler"
            sh "mvn test"
        }

        stage('Sonarqube Analysis') {
             //sh "mvn clean test"
             //sh "mvn clean test -Djacoco.skip=true"
             echo "Current branch: ${env.BRANCH_NAME}"
            withSonarQubeEnv('SonarQubeLocalServer') {

      /*  sh '''
            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar \
            -Dintegration-tests.skip=true \
            -Dmaven.test.failure.ignore=true \
            -Dsonar.projectKey=tech.zerofiltre.testing:calculator \
            -Dsonar.host.url=http://109.176.198.187:9000
        '''*/

             //sh " mvn sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true \"-Dsonar.projectKey=tech.zerofiltre.testing:calculator\" \"-Dsonar.host.url=http://109.176.198.187:9000\""
             sh "mvn sonar:sonar -s .m2/settings.xml \"-Dsonar.projectKey=tech.zerofiltre.testing:calculator\""
               
            }
            timeout(time: 1, unit: 'MINUTES') {
                def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }
        }

        stage("Image Prune") {
            imagePrune(CONTAINER_NAME)
        }
         stage('Build and package') {
            //sh "mvn -X clean compile 2>&1 | grep -i compiler"
            sh "mvn clean package"
        }
        stage('Image Build') {
            imageBuild(CONTAINER_NAME, CONTAINER_TAG)
        }

        stage('Push to Docker Registry') {
            withCredentials([usernamePassword(credentialsId: 'dockerhubcredential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
            }
        }

        stage('Run App') {
            withCredentials([usernamePassword(credentialsId: 'dockerhubcredential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                runApp(CONTAINER_NAME, CONTAINER_TAG, USERNAME, HTTP_PORT, ENV_NAME)

            }
        }

    } finally {
        deleteDir()
        //sendEmail(EMAIL_RECIPIENTS);
    }

}

def imagePrune(containerName) {
    try {
        sh "docker stop $containerName"
        sh "docker rm -f $containerName"
        sh "docker image prune -f"
    } catch (ignored) {
        echo "Container $containerName not found or already stopped"
    }
}

def imageBuild(containerName, tag) {
    sh "docker build -t $containerName:$tag  -t $containerName --pull --no-cache ."
    echo "Image build complete"
}

def pushToImage(containerName, tag, dockerUser, dockerPassword) {
    sh "docker login -u $dockerUser -p $dockerPassword"
    sh "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    sh "docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, dockerHubUser, httpPort, envName) {
    sh "docker pull $dockerHubUser/$containerName:$tag"
    sh "docker run --network springmysql-net --env SPRING_ACTIVE_PROFILES=$envName -d -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
 // sh "docker run --rm --env SPRING_ACTIVE_PROFILES=$envName -d -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}

def sendEmail(recipients, checkstyleReport = null) {
    def status = currentBuild.result ?: 'SUCCESS'
    def emoji = status == 'SUCCESS' ? '✅' : '❌'
    
    def reportContent = ""
    if (checkstyleReport) {
        reportContent = """
            
            📊 RAPPORT CHECKSTYLE:
            ${checkstyleReport}
        """
    }
    
    mail(
        to: recipients,
        subject: "${emoji} Build ${env.BUILD_NUMBER} - ${status} - (${currentBuild.fullDisplayName})",
        body: """
            Statut: ${status}
            Branche: ${env.BRANCH_NAME}
            Environnement: ${ENV_NAME}
            
            Consultez la console: ${env.BUILD_URL}/console
            
            Rapport Checkstyle: ${env.BUILD_URL}artifact/target/site/checkstyle.html
            Rapport de couverture: ${env.BUILD_URL}/jacoco
            Rapport SonarQube: http://109.176.198.187:9000${reportContent}
        """
    )
}

/*def sendEmail(recipients) {
    mail(
            to: recipients,
            subject: "Build ${env.BUILD_NUMBER} - ${currentBuild.currentResult} - (${currentBuild.fullDisplayName})",
            body: "Check console output at: ${env.BUILD_URL}/console" + "\n")
}*/

String getEnvName(String branchName) {
    if (branchName == 'main') {
        return 'prod'
    } else if (branchName.startsWith("release-") || branchName.startsWith("hotfix-") || branchName == 'ready') {
        return 'uat'
    }
    return 'dev'
}

String getHTTPPort(String branchName) {
    if (branchName == 'main') {
        return '9001'

    } else if (branchName.startsWith("release-") || branchName.startsWith("hotfix-") || branchName == 'ready') {
        return '9002'
    }
    return '9003'
}

String getTag(String buildNumber, String branchName) {
    if (branchName == 'main') {
        return buildNumber + '-unstable'
    }
    return buildNumber + '-stable'
}


