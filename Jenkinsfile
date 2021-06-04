pipeline{
    agent any
    tools {
        maven 'MAVEN'
    }
    stages {
        stage('Build Maven') {
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'GIT_REPO', url: 'https://github.com/subhankar720/docker_jenkins.git]]])

                sh "mvn -Dmaven.test.failure.ignore=true clean package"
                
            }
        }
        
    }
}