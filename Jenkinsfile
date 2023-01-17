pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerlogin')
  }

  tools { 
        ///depentencias 
        maven 'Maven 3.5.2' 

    }


// Stages.
  stages {   


stage('GIT CLONE') {
  steps {
                // Get code from a GitHub repository
    git url: 'https://github.com/BrunoSantos88/Jenkins-backend.git', branch: 'main',
    credentialsId: 'jenkins-aws'
          }
  }

stages{
    stage('SonarCloud-GateCode-Quality') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=Jenkins-backend -Dsonar.organization=brunosantos881388 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=53605ca03976b7b9426745604501d6d6914fc92a'
			}
        } 
}

   
stage('Synk-GateSonar-Security') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
}

///DockerProcesso
    stage('Docker Build') {
      steps {
        sh 'docker build -t brunosantos88/awsbackend backend/.'
      }
    }

    stage('Docker Login') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
   
    stage('Docker Push') {
      steps {
        sh 'docker push brunosantos88/awsbackend:latest'
      }
    }
  
  stage('Kubernetes Deployment backend') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh ('kubectl apply -f backend.yaml --namespace=developer')
		}
	      }
   	}

     stage ('Aguardar 180s Instalar OWSZAP'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
stage('OWSZAP Backend') {
    steps {
		withKubeConfig([credentialsId: 'kubelogin']) {
		sh('zap.sh -cmd -quickurl http://$(kubectl get services/backend --namespace=developer -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
	  archiveArtifacts artifacts: 'zap_report.html'
		}
	  }
    } 

  }
}