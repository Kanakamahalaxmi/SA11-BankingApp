pipeline {

    agent { label 'slave1' }
	
    tools {
        maven "maven-3.6.3"
    }

	environment {	
		DOCKERHUB_CREDENTIALS=credentials('dockerloginid')
	}
	
    stages {
        stage('SCM_Checkout') {
            steps {
                echo 'Perform SCM Checkout'
				git 'https://github.com/Kanakamahalaxmi/SA11-BankingApp.git'
            }
        }
        stage('Application_Build') {
            steps {
                echo 'Perform Maven Build'
				sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
			  post {
				failure {
				  sh "echo 'Send mail on failure'"
				  mail to:"mahalaxmikaranam@gmail.com", from: 'mahalaxmikaranam@gmail.com', subject:"FAILURE: ${currentBuild.fullDisplayName}", body: "Build failed."
				}
			  }
        }
        stage('Build Docker Image') {
            steps {
				sh "docker build -t lithika/bankapp-eta-app:V${BUILD_NUMBER} ."
				sh 'docker image list'
				sh "docker tag lithika/bankapp-eta-app:V${BUILD_NUMBER} lithika/bankapp-eta-app:latest"
            }
              post {
                success {
                  sh "echo 'Send mail docker Build Success'"
                  mail to:"mahalaxmikaranam@gmail.com", from: 'mahalaxmikaranam@gmail.com', subject:"App Image Created Please validate", body: "App Image Created Please validate - lithika/bankapp-eta-app:latest"
                }
                failure {
                  sh "echo 'Send mail docker Build failure'"
                  mail to:"mahalaxmikaranam@gmail.com", from: 'mahalaxmikaranam@gmail.com', subject:"FAILURE: ${currentBuild.fullDisplayName}", body: "Image Build failed."
                }
              }	
        }
        stage('Approve - push Image to dockerhub'){
            steps{
                
                //----------------send an approval prompt-------------
                script {
                   env.APPROVED_DEPLOY = input message: 'User input required Choose "Yes" | "Abort"'
                       }
                //-----------------end approval prompt------------
            }
        }
		stage('Login2DockerHub') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
		stage('Publish_to_Docker_Registry') {
			steps {
				sh "docker push lithika/bankapp-eta-app:latest"
			}
		}
		stage('Deploy to Kubernetes_Cluster') {
			steps {
				script {
					sshPublisher(publishers: [sshPublisherDesc(configName: 'Kubernetes', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'kubectl apply -f k8sdeployment.yaml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])	
				}
			}
		}
    }
}									
