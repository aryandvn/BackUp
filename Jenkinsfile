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
        stage('Creating Backup') {
            steps {
                sh """
                cd /
                git clone https://github.com/aryandvn/BackUp-Volume.git
                cd BackUp-Volume
                rm -rf *
                mkdir JenBackup
                mkdir SonarQubeBackup
                cd /
                cd var/jenkins_home/
                cp -r * /BackUp-Volume/JenBackup/
                ls -l
                docker cp 89e8fcfeda9b:/nexus-data /BackUp-Volume/
                docker cp 6a88f6322ca7:/opt/sonarqube/conf /BackUp-Volume/SonarQubeBackup
                docker cp 6a88f6322ca7:/opt/sonarqube/extensions /BackUp-Volume/SonarQubeBackup
                docker cp 6a88f6322ca7:/opt/sonarqube/data /BackUp-Volume/SonarQubeBackup
                docker cp 6a88f6322ca7:/opt/sonarqube/lib/bundled-plugins /BackUp-Volume/SonarQubeBackup
                cd /
                cd /BackUp-Volume
                tar -czvf JenBackup.tar.gz JenBackup/
                tar -czvf NexusBackup.tar.gz nexus-data/
                tar -czvf SonarQubeBackup.tar.gz SonarQubeBackup/
                rm -rf JenBackup/
                rm -rf nexus-data/
                rm -rf SonarQubeBackup/
                git status
                git add .
                git commit -m "Made changes"
                """
            } 
        }
    }
}
