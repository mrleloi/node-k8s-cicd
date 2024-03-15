// git repository info
def gitRepository = 'https://github.com/mrleloi/node-k8s-cicd.git'
def gitBranch = 'master'

def gitRepositoryConfig = 'https://github.com/mrleloi/node-k8s-cicd-helmconfig.git'
def gitRepositoryConfigPushUrl = 'github.com/mrleloi/node-k8s-cicd-helmconfig.git'
def gitBranchConfig = 'main'
def helmRepo = "node-k8s-cicd"
def helmChart = "node-k8s-cicd"
def helmValueFile = "values.yaml"

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
            withCredentials([usernamePassword(credentialsId: 'githubCredential', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh """#!/bin/bash
                   [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
                   git clone ${gitRepositoryConfig} --branch ${gitBranchConfig}
                   cd ${helmRepo}
                   sed -i 's|  tag: .*|  tag: "${version}"|' ${helmValueFile}
                   git add . ; git commit -m "Update to version ${version}";git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${gitRepositoryConfigPushUrl}
                   cd ..
                   [[ -d ${helmRepo} ]] && rm -r ${helmRepo}
                   """	
            }
          }
        }
    }
}
