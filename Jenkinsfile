pipeline {
    agent any

    environment {
        DOCKER_REGISTRY_CREDS = 'dockerhub'
        EMAIL_NOTIFICATION = 'khurramilyas37101@gmail.com'
        DOCKER_BFLASK_IMAGE = 'khurramilyas37101/demoflaskapp' // 
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t my-flask-app .'
                sh "docker tag my-flask-app ${DOCKER_BFLASK_IMAGE}"
            }
        }

        stage('Test') {
            steps {
                sh 'docker run my-flask-app python -m pytest app/tests/'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                        sh "docker push ${DOCKER_BFLASK_IMAGE}"

                        sh "docker pull ${DOCKER_BFLASK_IMAGE}:latest"
                        sh "docker stop my-flask-app || true"
                        sh "docker rm my-flask-app || true"

                        try {
                            sh "docker run -d -p 80:5000 --name my-flask-app ${DOCKER_BFLASK_IMAGE}:latest"
                        } catch (Exception e) {
                            currentBuild.result = 'FAILURE'
                            error("Deployment failed. Rolling back to the previous version.")
                        }
                    }
                }
            }
        }
    }

    post {
        failure {
            emailext body: 'The deployment of Flask App failed. Please check the Jenkins console for details.',
                     subject: 'Flask App Deployment Failed',
                     to: "${EMAIL_NOTIFICATION}"
                    }

        always {
            sh 'docker logout'
        }
    }
}
