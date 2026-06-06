pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git branch: 'master', url: 'https://github.com/briss/unir_devops-cloud_cp1_aws.git'
                sh 'curl -o samconfig.toml https://raw.githubusercontent.com/briss/unir_devops-cloud_cp1_aws_config/production/samconfig.toml'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region us-east-1
                    sam deploy \
                        --config-env production \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
            steps {
                script {
                    def baseUrl = sh(
                        script: '''
                            aws cloudformation describe-stacks \
                                --stack-name todo-list-aws-production \
                                --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                                --region us-east-1 \
                                --output text
                        ''',
                        returnStdout: true
                    ).trim()

                    sh "BASE_URL=${baseUrl} pytest --junitxml=result-rest.xml test/integration/todoApiTest.py -v -m readonly"
                }
                junit 'result-rest.xml'
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}
