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
