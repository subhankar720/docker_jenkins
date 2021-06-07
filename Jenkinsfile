pipeline{
    environment {

        registry = "subhankar720/docker_jenkins"

        registryCredential = 'subhankar720'
    }
    agent any
    tools {
        maven 'MAVEN'
    }
    stages {
        stage('Build Maven') {
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'subhankar720', url: 'https://github.com/subhankar720/docker_jenkins.git']]])

                sh "mvn -Dmaven.test.failure.ignore=true clean package"
                
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                  sh 'docker build -t subhankar720/docker_jenkins:latest .'
                }
            }
        }
        stage('Deploy Docker Image') {
            steps {
                script {
                 withCredentials([string(credentialsId: 'docker_password', variable: 'docker_password')]) {
                     docker login -u subhankar720 -p $docker_password
                 } 
                 sh 'docker push subhankar720/docker_jenkins:latest'
                }
            }
        }
    }
    
}