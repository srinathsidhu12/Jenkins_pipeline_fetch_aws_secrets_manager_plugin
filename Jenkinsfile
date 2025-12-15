pipeline {
    agent any

    environment {
        IMAGE_NAME = "srinathsidhu12/parallel_pipeline_java_app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {
        //Pulls application source code from Git repository
        stage('Checkout') {                                    
            steps {
                git branch: 'main',
                    url: 'https://github.com/yourusername/java-parallel-pipeline-demo.git'
            }
        }

        stage('Build & Test') {
            parallel {

                stage('Compile') {
                    steps {
                        sh 'mvn clean compile'     // Cleans previous build artifacts and compiles the source code
                    }
                }

                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'      //Executes unit test cases using Surefire plugin
                    }
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'     //Packages the application into JAR/WAR without executing tests
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'     //Creates Docker image using Dockerfile.
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin     //Login to DockerHub securely
                    docker push $IMAGE_NAME:$IMAGE_TAG     //Push image to remote registry
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker rm -f java-app || true      //Remove old container if it exists
                docker run -d -p 8080:8080 --name java-app $IMAGE_NAME:$IMAGE_TAG     // Run new container
                '''
            }
        }
    }
}

