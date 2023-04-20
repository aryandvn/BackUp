pipeline {
  agent any

  tools {
      maven "MAVEN"
  }

  stages {
    stage('Git Checkout') {
        steps {
            git branch: 'main', url: 'https://github.com/aryandvn/BackUp.git'
        }
    }
    stage('Static Analysis (SonarQube)') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withSonarQubeEnv('sq1') {
                        sh """
                        mvn clean verify sonar:sonar \
  			-Dsonar.projectKey=LoginWebApp \
  			-Dsonar.projectName='LoginWebApp' \
  			-Dsonar.host.url=http://10.12.124.93:9000 \
  			-Dsonar.token=sqp_26e41a15638fe35c488ac1e1a6282be6a69b1535
                        """
                    }
                }
            }
        }
    stage("Quality gate") {
            steps {
		    #Quality gates
                waitForQualityGate abortPipeline: true
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
    stage('Docker Build and Tag') {
        steps {
            sh 'docker build -t samplewebapp:latest .' 
            sh 'docker tag samplewebapp aryandvn24/samplewebapp:latest'
        }
    }

    stage('Publish image to Docker Hub') {
        steps {
         withDockerRegistry([ credentialsId: "DockerHub", url: "" ]) {
            sh  'docker push aryandvn24/samplewebapp:latest'
            }
        }
    }
    stage('Run Docker container on Jenkins Agent') {
         steps {
             script {
                def containerName = "WebApp"
                def isRunning = sh(
                    returnStatus: true,
                    script: "docker ps --filter \"name=${containerName}\" --format '{{.Names}}' | grep -w ${containerName}"
                )
                if (isRunning == 0) {
                    println "Container ${containerName} is running"
                    sh "docker stop ${containerName}"
                    sh "docker rm ${containerName}"
                    sh "docker rmi samplewebapp:latest"
                    sh "docker rmi aryandvn24/samplewebapp"
                    sh "docker run -d --name WebApp -p 8003:8080 aryandvn24/samplewebapp"
                } else {
                    sh "docker run -d --name WebApp -p 8003:8080 aryandvn24/samplewebapp"
                }
            }
        }
    }
  }
  post {
		always{
			sh """
				cd / && rm -rf UseCase/ && docker system prune -f
			"""
		}
    }
}
