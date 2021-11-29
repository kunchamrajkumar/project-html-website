node {

def IAMGE_NAME = params.IMAGE_NAME
def TAG_NAME = params.TAG_NAME
def ARTIFACTORY_IP = params.ARTIFACTORY_IP
def ARTIFACTORY_PORT = params.ARTIFACTORY_PORT
def ARTIFACTORY_KEY = params.ARTIFACTORY_KEY

stage('checkout') {
 git branch: 'master', credentialsId: 'gitlab_cred', url: 'https://gitlab.com/rajudruva2/maven-project2.git'   
 }

/*stage('build') {
    withMaven(jdk: 'java', maven: 'maven') {
        println "build is running"
        sh "mvn clean package"
    }
}*/

stage('create docker image') {
    sh """
        docker build -t ${IAMGE_NAME}:${TAG_NAME} .
    """
}



stage('push the image to artifactory') {
    withCredentials([usernamePassword(credentialsId: 'artifactory', passwordVariable: 'password', usernameVariable: 'username')]) {
    sh """
        docker login -u ${username} -p ${password} ${ARTIFACTORY_IP}:${ARTIFACTORY_PORT}/${ARTIFACTORY_KEY}
        docker tag ${IAMGE_NAME}:${TAG_NAME} ${ARTIFACTORY_IP}:${ARTIFACTORY_PORT}/${ARTIFACTORY_KEY}/${IAMGE_NAME}:${TAG_NAME}
        docker push ${ARTIFACTORY_IP}:${ARTIFACTORY_PORT}/${ARTIFACTORY_KEY}/${IAMGE_NAME}:${TAG_NAME}
        docker pull ${ARTIFACTORY_IP}:${ARTIFACTORY_PORT}/${ARTIFACTORY_KEY}/${IAMGE_NAME}:${TAG_NAME}
        

    """
}
}


stage("SSH Into k8s Server") {
        def remote = [:]
        remote.name = 'K8S master'
        remote.host = '10.0.0.6'
        remote.user = 'eqadmin'
        remote.password = 'hbRZgbGo#KHe'
        remote.allowAnyHosts = true

    stage('Put kuberenetes/maven-project2-deployment.yml onto k8smaster') {
        
            sshPut remote: remote, from: 'maven-project2-deployment.yml', into: '.'
            sshPut remote: remote, from: 'maven-project2-service.yml', into: '.'
        } 
    stage('Deploy to container') {
          sshCommand remote: remote, command: "kubectl apply -f <(istioctl kube-inject -f maven-project2-deployment.yml)"
          sshCommand remote: remote, command: "kubectl apply -f <(istioctl kube-inject -f maven-project2-service.yml)"

        }

     }


}

