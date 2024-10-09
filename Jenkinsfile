pipeline {
    agent any

    tools {
        nodejs 'node_20.13.1'
    }

    environment {
        S3_BUCKET_NAME = 'todolist-frontend-app'
        ELASTIC_BEANSTALK_APP_NAME = 'todolistapp'
        ELASTIC_BEANSTALK_ENV_NAME = 'Todolistapp-env'
        REGION = 'us-east-1'  // Modify as per your AWS region
    }

    stages {
        stage('Backend Build and Test') {
            steps {
                dir('todo-back-end') {
                    echo 'Setting executable permission for mvnw...'
                    sh 'chmod +x mvnw'

                    echo 'Building Spring Boot backend using Maven Wrapper...'
                    sh './mvnw clean install'

                    echo 'Running backend tests...'
                    sh './mvnw test'
                }
            }
        }

        stage('Frontend Build and Test') {
            steps {
                dir('todo-front-end') {
                    echo 'Installing dependencies...'
                    sh 'npm install'

                    echo 'Running frontend build...'
                    sh 'npm run build'

                    echo 'Running frontend tests...'
                    //sh 'npm test'
                }
            }
        }

        stage('Deploy Backend to AWS Elastic Beanstalk') {
            steps {
                dir('todo-back-end/target') {
                    script {
                        echo 'Deploying backend to AWS Elastic Beanstalk...'

                        // Prepare JAR file
                        def jarFile = sh(script: "ls *.jar", returnStdout: true).trim()
                        echo "JAR file found: ${jarFile}"

                        // Deploy to Elastic Beanstalk using AWS credentials
                        withAWS(credentials: 'aws-credentials-id', region: "${REGION}") {
                            sh '''
                            echo "Creating application version..."
                            aws elasticbeanstalk create-application-version --application-name ${ELASTIC_BEANSTALK_APP_NAME} --version-label ${BUILD_NUMBER} --source-bundle S3Bucket=${S3_BUCKET_NAME},S3Key=backends/${jarFile}

                            echo "Updating environment..."
                            aws elasticbeanstalk update-environment --application-name ${ELASTIC_BEANSTALK_APP_NAME} --environment-name ${ELASTIC_BEANSTALK_ENV_NAME} --version-label ${BUILD_NUMBER}
                            '''
                        }
                    }
                }
            }
        }

        stage('Deploy Frontend to AWS S3') {
            steps {
                dir('todo-front-end/dist') {
                    script {
                        echo 'Deploying frontend to AWS S3...'

                        // Sync the dist/ directory to S3 bucket using AWS credentials
                        withAWS(credentials: 'aws-credentials-id', region: "${REGION}") {
                            sh '''
                            echo "Syncing to S3..."
                            aws s3 sync . s3://${S3_BUCKET_NAME}/ --delete
                            '''
                        }
                    }
                }
            }
        }
    }
}
