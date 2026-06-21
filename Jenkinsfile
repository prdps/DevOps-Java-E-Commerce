pipeline {

    agent any

    tools {
        maven 'maven'
        jdk 'jdk-17'
    }

    options {

        timestamps()

        disableConcurrentBuilds()

        buildDiscarder(
            logRotator(
                numToKeepStr: '10',
                daysToKeepStr: '14'
            )
        )
    }

    environment {

        APP_NAME     = "maven-e-com-web-app"
        DOCKER_IMAGE = "pradeepsuresh/maven-e-com-web-app"
        IMAGE_TAG    = "${BUILD_NUMBER}"

    }

    stages {

        stage('Clean Workspace') {
            steps {

              //  cleanWs()
                echo 'completed'
            }
        }

        stage('Checkout Code') {
            steps {

                echo 'Checking out source code...'

                git(
                    branch: 'master',
                    url: 'https://github.com/prdps/DevOps-Java-E-Commerce.git'
                )

            }
        }

        stage('Build Application') {
            steps {

                echo 'Building Maven application...'
                sh 'java --version'
                sh 'whoami'
                sh 'mvn clean package -DskipTests'

            }
        }

        stage('Run Unit Tests') {
            steps {

                echo 'Running unit tests...'
                sh 'mvn test'

            }
        }

        stage('Build Docker Image') {
            steps {

                echo 'Building Docker image...'
                sh 'pwd && ls -la'
                sh """
                docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                """

            }
        }

        stage('Push Docker Image') {
            steps {

                echo 'Pushing Docker image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
        sh """
                echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
            """

            sh """
                docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
            """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {

                echo 'Deploying application to Kubernetes...'
                sh 'whoami && kubectl config current-context && kubectl get nodes'
                sh """
                sed -i "s|image: .*|image: pradeepsuresh/maven-e-com-web-app:${BUILD_NUMBER}|g" k8s-Deployment.yml
                """
                sh 'cat k8s-Deployment.yml'
                sh 'kubectl apply -f k8s-Deployment.yml'

            }
        }

        stage('Verify Deployment') {
            steps {

                echo 'Verifying Kubernetes deployment...'

                sh 'kubectl get pods'

                sh 'kubectl get svc'

            }
        }
    }

    post {

        success {

            echo 'Pipeline executed successfully.'

        }

        failure {

            echo 'Pipeline execution failed.'

        }

        always {

            script {

                try {

                    //cleanWs()
                    echo 'completed!!'

                } catch (Exception e) {

                    echo "Workspace cleanup skipped: ${e}"

                }
            }
        }
    }
    
}