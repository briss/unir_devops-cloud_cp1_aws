pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/briss/unir_devops-cloud_cp1_aws.git'
            }
        }

        stage('Static Tests') {
            parallel {
                stage('Flake8') {
                    sh 'flake8 --format=pylint --exit-zero src > flake8.out'
                    recordIssues(
                        tools:[flake8(name: 'Flake8', pattern: 'flake8.out')]
                    )
                }

                stage('Bandit') {
                    sh 'bandit --exit-zero -r src -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"'
                    recordIssues(
                        tools:[pyLint(name: 'Bandit', pattern: 'bandit.out')]
                    )
                }
            }
        }
    }
}
