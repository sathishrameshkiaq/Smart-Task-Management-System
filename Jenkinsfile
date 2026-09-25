pipeline {

    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'

        S3_BUCKET = 'sat-react-app-bucket'
        CLOUDFRONT_DISTRIBUTION_ID = 'E1CS5A31ZQEM6U'

        EC2_HOST = '13.204.68.109'
        EC2_USER = 'ubuntu'
        EC2_PROJECT_DIR = '/home/ubuntu/Smart-Task-Management-System'
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    branch: 'main',
                    credentialsId: 'Github-ID',
                    url: 'https://github.com/sathishrameshkiaq/Smart-Task-Management-System.git'
                )
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withCredentials([
                        string(
                            credentialsId: 'sonar-token',
                            variable: 'SONAR_TOKEN'
                        )
                    ]) {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=Smart-Task \
                                -Dsonar.sources=. \
                                -Dsonar.exclusions="**/node_modules/**,**/dist/**,**/build/**,**/.git/**" \
                                -Dsonar.host.url=http://localhost:9000 \
                                -Dsonar.token=\${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Install Frontend Dependencies') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Build React Application') {
            steps {
                dir('frontend') {
                    sh 'npm run build'
                }
            }
        }

        stage('Upload Frontend to S3') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'AccesskeyID',
                        variable: 'AWS_ACCESS_KEY_ID'
                    ),
                    string(
                        credentialsId: 'SECRET_ACCESS_KEY',
                        variable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        aws s3 sync \
                            frontend/dist/ \
                            s3://${S3_BUCKET}/ \
                            --delete
                    '''
                }
            }
        }

        stage('CloudFront Invalidation') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'AccesskeyID',
                        variable: 'AWS_ACCESS_KEY_ID'
                    ),
                    string(
                        credentialsId: 'SECRET_ACCESS_KEY',
                        variable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        aws cloudfront create-invalidation \
                            --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} \
                            --paths "/*"
                    '''
                }
            }
        }

        stage('Deploy Backend to EC2') {
            steps {
                sshagent(credentials: ['ec2-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no \
                            ${EC2_USER}@${EC2_HOST} '
                            cd ${EC2_PROJECT_DIR}

                            docker compose up -d --build

                            docker compose ps
                            '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }

        failure {
            echo 'Deployment failed. Check the Jenkins console output.'
        }
    }
}
