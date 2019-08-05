def label = "worker-${UUID.randomUUID().toString()}"
def project = 'heifetzwu'
def appName = 'ironman'
def tag = "v_${env.BUILD_NUMBER}"
// def img = "gcr.io/${project}/${appName}-${env.BRANCH_NAME}"
def img = "${project}/${appName}-${env.BRANCH_NAME}"
def imgWithTag = "${img}:${tag}"

def devNamespace = 'develop'
def proNamespace = 'production'

def GOOGLE_APPLICATION_CREDENTIALS    = '/home/jenkins/dev/secret'

podTemplate(label: label, containers: [
  
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'gcr.io/cloud-builders/kubectl', command: 'cat', ttyEnabled: true)
  
],
volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    secretVolume(secretName: 'jenkinsgcp', mountPath: '/home/jenkins/dev')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    env.GOOGLE_APPLICATION_CREDENTIALS=GOOGLE_APPLICATION_CREDENTIALS
    
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
              docker images
              docker push ${imgWithTag}
              """
          }
      }
    }
    stage('deploy') {
      container('kubectl') {  
        sh("sed -i.bak s#gcr.io/ithome-image#${imgWithTag}# ./k8s/deploy.yaml")
        switch (env.BRANCH_NAME) {
          case "master":
              sh("### echo test kubectl")
              sh("kubectl version")
              sh("gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}")
              // sh("gcloud init")
              sh("gcloud auth list")
              sh("gcloud container clusters get-credentials myck --zone us-central1-a --project plexiform-pilot-137623")
              sh("kubectl get pods")

              // replace namespace settings
              sh("sed -i.bak 's#env: current#env: ${devNamespace}#' ./k8s/service.yaml")
              sh("sed -i.bak 's#env: current#env: ${devNamespace}#' ./k8s/deploy.yaml")
              sh("kubectl --namespace=${devNamespace} apply -f ./k8s/service.yaml")
              sh("kubectl --namespace=${devNamespace} apply -f ./k8s/deploy.yaml")
              break
          case "production":
              sh("gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}")
              sh("gcloud auth list")
              sh("gcloud container clusters get-credentials myck --zone us-central1-a --project plexiform-pilot-137623")
              // replace namespace settings
              sh("sed -i.bak 's#env: current#env: ${proNamespace}#' ./k8s/service.yaml")
              sh("sed -i.bak 's#env: current#env: ${proNamespace}#' ./k8s/deploy.yaml")
              sh("kubectl --namespace=${proNamespace} apply -f ./k8s/service.yaml")
              sh("kubectl --namespace=${proNamespace} apply -f ./k8s/deploy.yaml")
              break
        }
      }
    }
  }
}