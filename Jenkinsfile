pipeline{
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
        stage("Publish to Nexus Repository Manager") {

            steps {

                script {

                    pom = readMavenPom file: "pom.xml";

                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");

                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"

                    artifactPath = filesByGlob[0].path;

                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {

                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            
                            protocol: 'http',

                            nexusUrl: '13.232.64.181:8081',

                            groupId: 'pom.com.mycompany.app',

                            version: 'pom.1.0-SNAPSHOT',

                            repository: 'nexus-repo',

                            credentialsId: 'nexus_cred',

                            artifacts: [

                                [artifactId: 'pom.my-app',

                                classifier: '',

                                file: artifactPath,

                                type: pom.packaging],

                                [artifactId: 'pom.my-app',

                                classifier: '',

                                file: "pom.xml",

                                type: "pom"]

                            ]

                        );

                    } else {

                        error "*** File: ${artifactPath}, could not be found";

                    }

                }

            }

        }
        stage('SonarQube Analysis') {
            steps{
                script{
                    def scannerHome = tool "SonarQube";
                        withSonarQubeEnv('SonarQube') {
                        sh """/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQube/bin/sonar-scanner \
                            -D sonar.projectVersion=1.0-SNAPSHOT \
                            -D sonar.login=admin \
                            -D sonar.password=admin \
                            -D sonar.projectBaseDir=/var/lib/jenkins/workspace/complete-pipeline-project \
                            -D sonar.projectKey=my-app \
                            -D sonar.sourceEncoding=UTF-8 \
                            -D sonar.language=java \
                            -D sonar.sources=src/main \
                            -D sonar.tests=src/test \
                            -D sonar.host.url=http://15.206.88.103:9000/"""
        }
    }
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
                     sh 'docker login -u subhankar720 -p $docker_password'
                      sh 'docker push subhankar720/docker_jenkins:latest'
                 } 
                 
                }
            }
        }
        stage('Deploy App on k8s') {
                steps {
                    sshagent(['k8s']) {
                        sh "scp -o StrictHostKeyChecking=no nodejs-app.yaml ubuntu@13.233.94.122:/home/ubuntu"
                        script {
                                try{
                                    sh "ssh ubuntu@13.233.94.122 kubectl apply -f ."
                                }
                                catch(error){
                                    sh "ssh ubuntu@13.233.94.122 kubectl create -f ."
                                }
                            }
                        }
      
                    }
                }

    
  }
}