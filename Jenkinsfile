pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'Jdk17'
    }

    environment {
        APP_PORT = '8081'
	DOCKER_IMAGE = "chohdigsaeir/springpetclinic"
        DOCKER_TAG   = "latest"
    }

    stages {

        stage('START') {
            steps {
                echo 'Pipeline started'
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

        stage('START APPLICATION') {
            steps {
                sh '''
                nohup mvn spring-boot:run \
                  -Dspring-boot.run.arguments=--server.port=${APP_PORT} \
                  > app.log 2>&1 &
                '''
            }
        }

        stage('WAIT FOR APP') {
            steps {
                sh '''
                echo "Waiting for app..."
                for i in {1..30}; do
                  curl -s http://localhost:${APP_PORT} && break
                  sleep 2
                done
                '''
            }
        }

        stage('SELENIUM TESTS') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SONARQUBE ANALYSIS') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
	
	stage('DOCKER BUILD') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

	stage('DOCKER PUSH') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('STOP APPLICATION') {
            steps {
                sh 'pkill -f spring-boot || true'
            }
        }

        stage('END') {
            steps {
                echo 'Pipeline finished'
            }
        }
    }
}

