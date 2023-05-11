pipeline{
agent any
environment{
    VERSION="${env.BUILD_ID}"
}
stages{
stage('soanr quality status'){
    agent{
        docker{
            image 'maven'
        }
    }
    steps{
        script{
          withSonarQubeEnv(credentialsId: 'sonar-token') {
            sh 'mvn clean package sonar:sonar'
          }
        }
    }
}

stage('Quality Gate status'){
    steps{
        script{
            waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        }
    }
}

stage('dockerbuild & docker push to Nexus repo'){
    steps{
        script{
            withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_creds')]) {
            sh '''
              docker build -t 54.196.193.5:8083/springapp:${VERSION} .

               docker login -u admin -p $nexus_creds 54.196.193.5:8083

               docker push 54.196.193.5:8083/springapp:${VERSION}
               
               docker rmi 54.196.193.5:8083/springapp:${VERSION}


            '''

        }
    }
}



}

}
post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "eapenmani@gmail.com";  
		}
	}

}