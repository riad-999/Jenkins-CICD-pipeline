pipeline {
    agent any
    environment {
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = "087380772019.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://087380772019.dkr.ecr.us-east-1.amazonaws.com"
        cluster = "vprofile_cluster"
        service = "vprofileappsvc"
    }
  stages {
    stage('Fetch code'){
      steps {
        script {
            // Define the SSH key credentials ID configured in Jenkins
            def sshKeyCredentials = 'githubkey'

            // Checkout code from the private GitHub repository using SSH
            git credentialsId: sshKeyCredentials, branch: 'main', url: 'git@github.com:riad-999/Jenkins-CICD.git'
        }
    }
    }


    stage('Test'){
      steps {
        sh 'mvn test'
      }
    }

    stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('build && SonarQube analysis') {
            environment {
             scannerHome = tool 'sonar4.7'
          }
            steps {
                withSonarQubeEnv('sonar') {
                 sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

    stage('Build App Image') {
       steps {
       
         script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
             }

     }
    
    }

    stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
     }
     
     stage('Deploy to ecs') {
          steps {
		        withAWS(credentials: 'awscreds', region: 'us-east-1') {
		          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
		        }
		      }
		   }
     
  }
}
