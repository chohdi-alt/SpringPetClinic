pipeline {
	agent any

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
				sh 'mvn clean test'
			}
		}
		stage('SOANRQUBE ANALYSIS') {
			steps {
				withSonarQubeEnv('SonarQube') {
					sh 'mvn sonar:sonar'
				}
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
