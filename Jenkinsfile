pipeline {
  agent any
	// Specify Maven tool installation
    tools {
        maven '3.6.0' 
	jdk 'JDK-11'
    }
	
  environment {
    DOCKERHUB_CREDENTIALS = credentials('docker-cred')
    REMOTE_SERVER = '13.57.179.83'
    REMOTE_USER = 'ubuntu' 	  	  
  }
	
  // Fetch code from GitHub
	
  stages {
    stage('checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Ganesh9082/opticalarc.git'

      }
    }
	  
   // Build Java application
	  
    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
      }
	    
     // Post building archive Java application
	    
      post {
        success {
          archiveArtifacts artifacts: '**/target/*.jar'
        }
      }
    }
	  
  // Test Java application
	  
    stage('Maven Test') {
      steps {
        sh 'mvn test'
      }
    }
	  
   // Build docker image in Jenkins
	  
    stage('Build Docker Image') {

      steps {
        sh 'docker build -t opticalarc:latest .'
        sh 'docker tag opticalproject ganeshshinde111/opticalarc:latest'
      }
    }
	  
   // Login to DockerHub before pushing docker Image
	  
    stage('Login to DockerHub') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u    $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
	  
   // Push image to DockerHub registry
	  
    stage('Push Image to dockerHUb') {
      steps {
        sh 'docker push ganeshshinde111/project:latest'
      }
      post {
        always {
          sh 'docker logout'
        }
      }

    }
	  
   // Pull docker image from DockerHub and run in EC2 instance 
	  
    stage('Deploy Docker image to AWS instance') {
      steps {
        script {
          sshagent(credentials: ['aws-cred']) {
          sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker stop javaApp || true && docker rm javaApp || true'"
	  sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker pull ganeshshinde111/opticalarc'"
          sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker run --name javaApp -d -p 8081:8081 ganeshshinde111/opticalarc'"
          }
        }
      }
    }
  }
}
