pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME ?: 'main'}", url: 'https://github.com/sakunthala-lash/Jenkins-pipeline.git', credentialsId: 'github-credentials'
            }
        }
        stage('Build') {
            steps {
                bat 'mvn clean install' 
            }
        }
        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            junit 'target/surefire-reports/*.xml'
        }
        success {
            step([
                $class: 'GitHubCommitStatusSetter',
                reposSource: [$class: "ManuallyEnteredRepositorySource", url: 'https://github.com/sakunthala-lash/Jenkins-pipeline.git'],
                contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "Jenkins CI"],
                statusResultSource: [$class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", state: "SUCCESS", message: "Build succeeded"]]]
            ])
        }
        failure {
            step([
                $class: 'GitHubCommitStatusSetter',
                reposSource: [$class: "ManuallyEnteredRepositorySource", url: 'https://github.com/sakunthala-lash/Jenkins-pipeline.git'],
                contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "Jenkins CI"],
                statusResultSource: [$class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", state: "FAILURE", message: "Build failed"]]]
            ])
        }
    }
}