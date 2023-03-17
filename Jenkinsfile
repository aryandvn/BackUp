pipeline {
    agent any

    tools {
        maven "MAVEN"
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '2015f835-5756-4d8f-86c2-c468cc526883', url: 'https://github.com/aryandvn/BackUp.git']])
            }
        }
        stage('Static Analysis (SonarQube)') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withSonarQubeEnv('sq1') {
                        sh """
                        mvn clean verify sonar:sonar -Dsonar.projectKey=Jenkins_Sonar 
                        """
                    }
                }
            }
        }
        stage('Build (war)') {
           steps {
             
                echo "Building war file"
                sh 'mvn package'             
          }
        }
        stage('Artifact Upload (Nexus)') {
            steps{
                nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'LoginWebApp', 
                            classifier: '', 
                            file: 'target/LoginWebApp-1.war', 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: 'Admin_Nexus', 
                    groupId: 'com.devops4solutions', 
                    nexusUrl: '10.12.124.93:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'DEMO-2', 
                    version: '1'
            }
        }
    }
}
