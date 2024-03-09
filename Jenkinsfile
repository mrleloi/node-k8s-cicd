// git repository info
def gitRepository = 'https://github.com/mrleloi/node-k8s-cicd.git'
def gitBranch = 'master'

// Image infor in registry
def imageGroup = 'leloimr'
def appName = "node-k8s-cicd"
def namespace = "helm-demo"	

// harbor-registry credentials
def registryCredential = 'dockerhub_leloimr'
// github credentials
def githubCredential = 'github_mrleloi'

dockerBuildCommand = './'
def version = "prod-0.${BUILD_NUMBER}"

pipeline {
    agent {
        kubernetes {
            label "jenkins-agent-dockernodejs"
        }
    }
    
    environment {
        HELM_APP_NAME = 'test1'
        HELM_CHART_DIRECTORY = "k8s/nodejs-k9s-cicd"
        DOCKER_IMAGE_NAME = "${imageGroup}/${appName}"
    }

    stages {
    
        stage('Checkout project') 
        {
          steps 
          {
            echo "checkout project"
            git branch: gitBranch,
               credentialsId: githubCredential,
               url: gitRepository
            sh "git reset --hard"
          }
        }
        stage('Build docker and push to registry') 
        {
          steps 
          {
            container('docker') {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME, dockerBuildCommand)
                    docker.withRegistry('', registryCredential ) {                       
                       app.push(version)
                    }
    
                    sh "docker rmi ${DOCKER_IMAGE_NAME}:${version} -f"				
                }
            }
          }
        }
        stage('Build helm') 
        {
          steps 
          {
            container('helm') {
                script {
                    sh "helm lint ./${HELM_CHART_DIRECTORY}"
                    sh "helm upgrade --wait --timeout 60 --set image.tag=${version} ${HELM_APP_NAME} ./${HELM_CHART_DIRECTORY}"		
                }
            }
          }
        }
    }
}
