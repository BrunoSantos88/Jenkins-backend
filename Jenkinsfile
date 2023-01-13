pipeline {
  agent any
  tools { 
        maven 'Maven 3.5.2'  
    }



stages{

stage('Slack Notification(Start)') {
      steps {
        slackSend message: 'Pipeline Inciada!. Necessidade de atenção, caso seja em Produção!'

}
}

stage('Clone repository') { 
      steps { 
        script{
          checkout scm
            }
             } 
    }
stage('Slack Notification(test unit code and vulnerability)') {
    steps {
      slackSend message: 'Pipeline está no estagio de teste no codigo. O Processo será realiazado no Quality Gate, são teste de Sonar e Synk, ambos vão verificar "bugs e vulnerabilidade" em nosso codigo!'

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
stage('Socker Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin88", url: ""]) {
                 script{
                 app =  docker.build("backend")
                 }
               }
            }
    }

stage('Docker Push') {
            steps {
                script{
                    docker.withRegistry('https://555527584255.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}

stage('Slack Notification(Dockerização)') {
    steps {
      slackSend message: 'Processo de Criar uma Release de imagem do Docker foi efetuado com sucesso!'

}
}
	    
  }


// Email Notification
post {
        always {
            echo "Notifying build result by email"
        }
success {
            mail to: 'infratidevops@gmail.com',
                 subject: "SUCCESS: ${currentBuild.fullDisplayName}",
                 body: "Pipeline passou, Efetou com Sucesso"

        }
failure {
           mail to: 'infratidevops@gmail.com',
                subject:"FAILURE: ${currentBuild.fullDisplayName}",
                body: "Pipeline Falhou , verificar os parametros corretos!"

        }
      }
}
