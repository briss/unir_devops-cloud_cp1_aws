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

        stage('Rest Test') {
            steps {
                script {
                    def baseUrl = sh(
                        script: '''
                            aws cloudformation describe-stacks \
                                --stack-name todo-list-aws-staging \
                                --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                                --region us-east-1 \
                                --output text
                        ''',
                        returnStdout: true
                    ).trim()

                    sh "BASE_URL=${baseUrl} pytest --junitxml=result-rest.xml test/integration/todoApiTest.py -v"
                }
                junit 'result-rest.xml'
            }
        }

        stage('Promote') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'github-credentials-devopscloud',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN')
                ]) {
                    unstash name: 'code'
                    sh '''
                        git config user.email "jenkins@bgs.dev"
                        git config user.name "Jenkins"
                        git checkout master
                        git merge develop
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/briss/unir_devops-cloud_cp1_aws.git master
                    '''
                }
            }
            post {
                always {
                    deleteDir()
                }
            }
        }
    }
}
