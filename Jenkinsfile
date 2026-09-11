pipeline {

    agent any

    environment {

        AWS_REGION = 'ap-south-1'

        S3_BUCKET = 'spring-boot-lambda-deployment-audit'

        LAMBDA_FUNCTION = 'springboot-lambda'

        JAR_FILE = 'target/spring-boot-security-postgresql-1.0.0-SNAPSHOT-aws.jar'

        S3_KEY = "lambda/springboot-app/build-${BUILD_NUMBER}.jar"
    }

    stages {

        stage('Checkout') {

            steps {

                echo 'Checking out source code...'

                checkout scm
            }
        }

        stage('Build') {

            steps {

                echo 'Building Spring Boot application...'

                bat 'mvnw.cmd clean package -DskipTests'
            }
        }

        // stage('Verify JAR') {

        //     steps {

        //         bat """
        //             if not exist "%JAR_FILE%" (
        //                 echo JAR file not found
        //                 exit /b 1
        //             )

        //             echo JAR found:
        //             dir "%JAR_FILE%"
        //         """
        //     }
        // }

        stage('Verify JAR') {
    steps {
        bat 'dir target'
    }
}

        stage('Upload JAR to S3') {

            steps {

                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-lambda-deployment']
                ]) {

                    bat """
                        aws s3 cp "%JAR_FILE%" ^
                        "s3://%S3_BUCKET%/%S3_KEY%" ^
                        --region %AWS_REGION%
                    """
                }
            }
        }

        stage('Deploy Lambda') {

            steps {

                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-lambda-deployment']
                ]) {

                    bat """
                        aws lambda update-function-code ^
                        --function-name "%LAMBDA_FUNCTION%" ^
                        --s3-bucket "%S3_BUCKET%" ^
                        --s3-key "%S3_KEY%" ^
                        --region "%AWS_REGION%" ^
                        --publish
                    """
                }
            }
        }
    }

    post {

        success {

            echo '===================================='
            echo 'Lambda deployment successful!'
            echo "Build: ${BUILD_NUMBER}"
            echo "S3 Key: ${S3_KEY}"
            echo '===================================='
        }

        failure {

            echo 'Lambda deployment failed.'
        }
    }
}
