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
            env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
        }

        stage('Checkout') {
            checkout scm
        }

        stage('Build with test') {

            sh "mvn clean compile"
        }

        stage('Sonarqube Analysis') {
             sh "mvn clean compile test-compile"
             echo "Current branch: ${env.BRANCH_NAME}"
            withSonarQubeEnv('SonarQubeLocalServer') {
                sh " mvn sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true"
               
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
        sendEmail(EMAIL_RECIPIENTS);
    }

}

def imagePrune(containerName) {
    try {
        sh "docker image prune -f"
        sh "docker stop $containerName"
    } catch (ignored) {
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
    sh "docker pull $dockerHubUser/$containerName"
    sh "docker run --rm --env SPRING_ACTIVE_PROFILES=$envName -d -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}

def sendEmail(recipients) {
    mail(
            to: recipients,
            subject: "Build ${env.BUILD_NUMBER} - ${currentBuild.currentResult} - (${currentBuild.fullDisplayName})",
            body: "Check console output at: ${env.BUILD_URL}/console" + "\n")
}

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
