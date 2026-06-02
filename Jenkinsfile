pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/briss/unir_devops-cloud_cp1_aws.git'
                stash name: 'code', includes: '**'
            }
        }

        stage('Static Tests') {
            parallel {
                stage('Flake8') {
                    steps {
                        unstash name: 'code'
                        sh 'flake8 --format=pylint --exit-zero src > flake8.out'
                        recordIssues(
                            tools:[flake8(name: 'Flake8', pattern: 'flake8.out')]
                        )
                    }
                    post {
                        always {
                            deleteDir()
                        }
                    }
                }

                stage('Bandit') {
                    steps {
                        unstash name: 'code'
                        sh 'bandit --exit-zero -r src -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}({severity})] {msg}"'
                        recordIssues(
                            tools:[pyLint(name: 'Bandit', pattern: 'bandit.out')]
                        )
                    }
                    post {
                        always {
                            deleteDir()
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                unstash name: 'code'
                sh '''
                    sam build
                    sam validate --region us-east-1
                    sam deploy \
                        --config-env staging \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                '''
            }
        }
    }
}
