pipeline {
    agent any
    tools {
     maven    "MAVEN"
    }
    
       environment {
        registry = "176626499739.dkr.ecr.us-west-2.amazonaws.com/myecrrepo"
    }
    

    stages {
        stage('GITHUB') {
            steps {
                git credentialsId: 'github', url: 'https://github.com/kumar63086/maven-web-application.git'
              
            }
        }
        stage("BUILD"){
            steps{
                sh "mvn clean package sonar:sonar"
            }
        }
        
        
       
 
      // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry
        }
      }
    }
    
   // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
         withEnv(["AWS_ACCESS_KEY_ID=${env.AWS_ACCESS_KEY_ID}", "AWS_SECRET_ACCESS_KEY=${env.AWS_SECRET_ACCESS_KEY}", "AWS_DEFAULT_REGION=${env.AWS_DEFAULT_REGION}"]) {
          sh "aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 176626499739.dkr.ecr.us-west-2.amazonaws.com"
            
                sh "docker build -t myecrrepo ."
                sh "docker tag myecrrepo:latest 176626499739.dkr.ecr.us-west-2.amazonaws.com/myecrrepo:latest"
                sh "docker push 176626499739.dkr.ecr.us-west-2.amazonaws.com/myecrrepo:latest"
         }
        }
      }
      
    }
       // Stopping Docker containers for cleaner Docker run
     stage('stop previous containers') {
         steps {
            sh 'docker ps -f name=mypythonContainer -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=mypythonContainer -q | xargs -r docker container rm'
         }
       }
       stage('Docker Run') {
     steps{
         script {
                sh 'docker run -d -p 8096:5000 --rm --name mypythonContainer 176626499739.dkr.ecr.us-west-2.amazonaws.com/myecrrepo:latest'
            }
      }
    }
        
    }
}
