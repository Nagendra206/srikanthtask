pipeline {
    agent any
     environment {
        registry = "502746322071.dkr.ecr.ap-south-1.amazonaws.com/demo-project"
    }
   
    stages {
          stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Nagendra206/srikanthtask.git'
            }
        }
           stage('Building image') {
             steps{
                  script {
                   dockerImage = docker.build registry
                   }
      }
           }
    
            stage('Pushing to ECR') {
             steps{  
                  script {
               withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws_cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 502746322071.dkr.ecr.ap-south-1.amazonaws.com'
     sh 'docker push 502746322071.dkr.ecr.ap-south-1.amazonaws.com/demo-project:myapp'
}

}
                  }
            }
             stage('stop previous containers') {
               steps {
            sh 'docker ps -f name=mypythonContainer -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=mypythonContainer -q | xargs -r docker container rm'
         }
       }
            stage('Docker Run') {
              steps{
                   script {
                sh 'docker run -d -p 3000:3000 --rm --name mypythonContainer 502746322071.dkr.ecr.ap-south-1.amazonaws.com/demo-project:myapp'     
      }
    }
        }
    }
  }
