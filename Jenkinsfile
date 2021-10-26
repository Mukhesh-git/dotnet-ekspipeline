pipeline {
  agent any
  triggers {
    pollSCM '* * * * *'
  }
  stages {
    stage('SonarQube Analysis') {
      steps {
        sh '''
        echo $PATH
	export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/home/ubuntu/.dotnet/tools
	dotnet sonarscanner begin /k:"Dotnet" /d:sonar.host.url=http://cmsiva.mukesh.website /d:sonar.login=51d9aa716cd5381458faa5f306f274fcccdf4bf4
	dotnet restore panz.csproj
	dotnet build panz.csproj -c Release
	dotnet sonarscanner end /d:sonar.login=51d9aa716cd5381458faa5f306f274fcccdf4bf4
        
        '''
      }
    }
    stage('Dotnet Publish') {
      steps {
        sh 'dotnet publish panz.csproj -c Release'
      }   
    }
   stage('Docker build and push') {
      steps {
        sh '''
      aws --version
      REPOSITORY_URI=160006709718.dkr.ecr.us-east-1.amazonaws.com/dotnet-app
      REPOSITORY_NAME=dotnet-app
      DOCKER_LOGIN_PASSWORD=$(aws ecr get-login-password  --region us-east-1)
      docker login -u AWS -p $DOCKER_LOGIN_PASSWORD https://160006709718.dkr.ecr.us-east-1.amazonaws.com/
      docker build -t dotnet-app:$BUILD_NUMBER .
      echo Build completed on `date`
      echo Pushing the Docker images...
      docker push $REPOSITORY_URI:dotnet-app-${BUILD_NUMBER}
	  '''
     }   
   }
    stage('ecs deploy') {
      steps {
        sh '''
          chmod +x changebuildnumber.sh
          ./changebuildnumber.sh $BUILD_NUMBER
	  sh -x ecs-auto.sh
          '''
     }    
    }
}
post {
    failure {
        mail to: 'unsolveddevops@gmail.com',
             subject: "Failed Pipeline: ${BUILD_NUMBER}",
             body: "Something is wrong with ${env.BUILD_URL}"
    }
     success {
        mail to: 'unsolveddevops@gmail.com',
             subject: "successful Pipeline:  ${env.BUILD_NUMBER}",
             body: "Your pipeline is success ${env.BUILD_URL}"
    }
}

}
