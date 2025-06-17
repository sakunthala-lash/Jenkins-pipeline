pipeline {
    agent any

    environment {
        // Define GitHub credentials for API calls
        GITHUB_TOKEN = credentials('github-credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout the current branch or PR
                git branch: "${env.BRANCH_NAME ?: 'main'}", url: 'https://github.com/sakunthala-lash/Jenkns-pipeline.git', credentialsId: 'github-credentials'
            }
        }
        stage('Get Commit SHA') {
            steps {
                // Capture the commit SHA for GitHub status updates
                script {
                    env.PR_COMMIT_SHA = bat(script: 'git rev-parse HEAD', returnStdout: true).trim().split('\r?\n')[-1].trim()
                }
            }
        }
        stage('Build') {
            steps {
               dir('demo') { 
                    bat 'mvn clean install'
                }
            }
        }
        stage('Test') {
            steps {
                dir('demo') { 
                      bat 'mvn test'
                }
              
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            junit 'target/surefire-reports/*.xml'
        }
        success {
            githubNotify('success', 'Build succeeded')
        }
        failure {
            githubNotify('failure', 'Build failed')
        }
    }
}

def githubNotify(String status, String description) {
    withCredentials([string(credentialsId: 'github-credentials', variable: 'GITHUB_TOKEN')]) {
        bat """
            curl -X POST -H "Authorization: token %GITHUB_TOKEN%" ^
            -H "Accept: application/vnd.github.v3+json" ^
            -d "{\\"state\\": \\"${status}\\", \\"description\\": \\"${description}\\", \\"context\\": \\"Jenkins CI\\"}" ^
            "https://api.github.com/repos/sakunthala-lash/Jenkins-pipeline/statuses/%PR_COMMIT_SHA%"
        """
    }
}
