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

        stage('Pushing image to DockerHub using AWS Secrets Manager') {
            steps {
              // withAWS block tells Jenkins to use the IAM Role
               withAWS(region: 'ap-south-1') {
                    script {
                        // Fetch the secret value from AWS Secrets Manager
                        // --secret-id : name of the secret in AWS
                        // --query SecretString : extracts only the actual secret data
                        // --output text : removes JSON metadata and returns clean output
                        def secretJson = sh(
                            script: '''
                            aws secretsmanager get-secret-value \
                             --secret-id dockerhub/credentials \
                             --query SecretString \
                             --output text
                            ''',
                            returnStdout: true      // Capture command output into a variable
                         ).trim()                  // Remove extra whitespace/newlines

                        // Convert the secret JSON string into a Groovy object
                        // Expected secret format:
                        // {
                        //   "username": "dockerhub_user",
                        //   "password": "dockerhub_password"
                        // }
                        def secret = readJSON text: secretJson

                        // Login to DockerHub securely
                        // Password is passed via STDIN (not visible in logs)
                        // This avoids exposing credentials in Jenkins console output
                        sh """
                          echo "${secret.password}" | docker login \
                             -u "${secret.username}" \
                             --password-stdin

                       // Push the Docker image (built earlier) to DockerHub
                       // Image tag uses Jenkins BUILD_NUMBER for versioning
                       docker push $IMAGE_NAME:$IMAGE_TAG
                       """
                  }
              } 
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

