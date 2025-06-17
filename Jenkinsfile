pipeline {
    agent any

    environment {
        GITHUB_REPO = 'https://github.com/sakunthala-lash/Jenkins-pipeline.git'
    }

    stages {
        stage('Extract Payload Data') {
            steps {
                script {
                        echo "PR Branch Name: ${PR_BRANCH_NAME}"
                        echo "PR Commit SHA: ${PR_COMMIT_SHA}"
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "refs/heads/${PR_BRANCH_NAME}"]],
                        userRemoteConfigs: [[url: "${GITHUB_REPO}"]]
                    ])
                }
            }
        }

        stage('List All Environment Variables') {
            steps {
                script {
                    echo "Listing all available environment variables:"
                    bat 'set'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    try {
                        dir('demo') {
                            bat 'mvn clean install'
                        }
                    } catch (Exception e) {
                        githubNotify('failure', 'Maven build failed. Check logs.')
                        throw e
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                githubNotify('success', 'Jenkins build and tests passed!')
            }
        }
        failure {
            script {
                githubNotify('failure', 'Build or tests failed. Fix before merging!')
            }
        }
    }
}

def githubNotify(String status, String description) {
    withCredentials([string(credentialsId: 'github-credentials', variable: 'GITHUB_TOKEN')]) {
        //def commitHash = bat(script: 'git rev-parse HEAD', returnStdout: true).trim().split("\r?\n")[-1].trim()

        bat """
            curl -X POST -H "Authorization: token %GITHUB_TOKEN%" ^
            -H "Accept: application/vnd.github.v3+json" ^
            -d "{\\"state\\": \\"${status}\\", \\"description\\": \\"${description}\\", \\"context\\": \\"Jenkins CI\\"}" ^
            "https://api.github.com/repos/sakunthala-lash/Jenkins-pipeline/statuses/${PR_COMMIT_SHA}"
        """
    }
}
