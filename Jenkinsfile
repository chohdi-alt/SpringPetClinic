pipeline {
	agent any
	
	tools {
		maven 'Maven'
		jdk'JDk17'
	}

	environment {
		APP_Port = '8081'
	}

	stages {
		stage('START') {
			steps {
				echo 'Pipelined started'
			}
		}
		
		stage('CHECKOUT') {
			steps {
				checkout scm
			}
		}

		stage('BUILD') {
			steps {
				sh 'mvn clean compile'
			}
		}
		stage('SOANRQUBE ANALYSIS') {
			steps {
				withSonarQubeEnv('SonarQube') {
					sh 'mvn sonar:sonar'
				}
			}
		}
		stage('START APPLICATION') {
			steps {
				sh '''
					nohup mcn spring-boot:run \
						-Dspring-boot.run.arguments=--server.port=8081 \
						> app.log 2>&1 & 
						slepp 25
				'''
			}
		}

		stage('SELENIUM TESTS') {
			steps {
				sh 'mvn test'
			}
		}

		stage('END') {
			steps {
				echo 'Pipeline finished successfully'
			}
		}
	}
	
	post {
		always {
			junit 'target/surefire-reports/*.xml'
		}
		failure {
			echo 'Pipeline failed'
		}
		success {
			echo 'Pipeline succeded'
		}
	}
}
