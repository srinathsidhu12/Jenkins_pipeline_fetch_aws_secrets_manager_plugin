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
                git branch: 'master',
                    url: 'https://github.com/srinathsidhu12/Jenkins_parallel_pipeline.git'
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

        stage('Push to DockerHub using aws secret manager credentials') {
            steps {
               script {
                    sh """
                        set +x  #disables command echoing for all sensitive operations.

                        # Fetch secret JSON
                        secretJson=\$(aws secretsmanager get-secret-value \
                          --secret-id my_dockerhub_cred \
                          --query SecretString --output text)

                        # Extract username and password
                        dockerUser=\$(echo "\$secretJson" | jq -r .username)
                        dockerPass=\$(echo "\$secretJson" | jq -r .password)

                        # Login and push
                        echo "\$dockerPass" | docker login -u "\$dockerUser" --password-stdin
                        docker push $IMAGE_NAME:$IMAGE_TAG

                        set -x
                    """
              }
           }
        }
        stage('Deploy Container') {
            steps {
                sh '''
                docker rm -f java-app || true      //Remove old container if it exists
                docker run -d -p 9090:8080 --name java-app $IMAGE_NAME:$IMAGE_TAG     // Run new container
                '''
            }
        }
    }
    post {
        success {
            mail to: 'srinathmuthyala99@gmail.com',
                 subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """
                 Hi Team,

                 Jenkins Job: ${env.JOB_NAME}
                 Build Number: ${env.BUILD_NUMBER}
                 Status: SUCCESS

                 Build URL:
                 ${env.BUILD_URL}

                 Regards,
                 Jenkins
                 """
        }
        failure {
            mail to: 'srinathmuthyala99@gmail.com',
                 subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: """
                 Hi Team,

                 Jenkins Job: ${env.JOB_NAME}
                 Build Number: ${env.BUILD_NUMBER}
                 Status: FAILED

                 Please check the logs:
                 ${env.BUILD_URL}

                 Regards,
                 Jenkins
                 """
        }
    }
}

