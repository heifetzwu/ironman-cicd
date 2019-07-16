def label = "worker-${UUID.randomUUID().toString()}"
def project = 'plexiform-pilot-137623'
def appName = 'ironmanapp'
def tag = "v_${env.BUILD_NUMBER}"
// def img = "gcr.io/${project}/${appName}-${env.BRANCH_NAME}"
def img = "${project}/${appName}-${env.BRANCH_NAME}"
def imgWithTag = "${img}:${tag}"

def devNamespace = 'develop'
def proNamespace = 'production'
podTemplate(label: label, containers: [
  
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  
],
volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
 
    
    stage('Create Docker images') {
      container('docker') {
          sh """
            docker build -t ${imgWithTag} .
            """

          withCredentials([[$class: 'UsernamePasswordMultiBinding',
            credentialsId: 'dockerhub',
            usernameVariable: 'DOCKER_HUB_USER',
            passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
            sh """
              echo "gitCommit="${gitCommit}
              docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
              # docker push namespace/my-image:${gitCommit}
              echo "### docker push ${imgWithTag}"
              docker push ${imgWithTag}
              """
          }
      }
    }
    
    
  }
}