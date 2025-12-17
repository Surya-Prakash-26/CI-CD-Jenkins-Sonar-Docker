pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "dockerhubusername/ec2-ci-cd"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh ''' 
		python3 -m venv venv
		. venv/bin/activate
		pip install -r requirements.txt
		'''		    
            }
        }

        stage('Test') {
            steps {
                sh ''' 
		. venv/bin/activate
		python3 test_app.py
		'''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=ec2-ci-cd \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    docker login -u $DOCKER_USER -p $DOCKER_PASS
                    docker build -t $DOCKER_IMAGE .
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop ec2-app || true
                docker rm ec2-app || true
                docker run -d --name ec2-app $DOCKER_IMAGE
                '''
            }
        }
    }
}

