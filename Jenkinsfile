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

stage('dockerbuild & docker push to Nexus repository'){
    steps{
        script{
            withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_creds')]) {
            sh '''
              docker build -t 3.90.88.236:8083/springapp:${VERSION} .

               docker login -u admin -p $nexus_creds 3.90.88.236:8083

               docker push 3.90.88.236:8083/springapp:${VERSION}
               
               docker rmi 3.90.88.236:8083/springapp:${VERSION}


            '''

        }
    }
}
}
stage('Identifying misconfigs using datree in helm chart'){
    steps{
        script{
            dir('kubernetes/myapp/') {
                withEnv(['DATREE_TOKEN=960ba681-237c-4a8c-abcb-5744fdd516c6']) {
                sh 'helm datree test .'
                }
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