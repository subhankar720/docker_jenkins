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
    }
    
}