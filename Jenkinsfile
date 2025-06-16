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
                dir('demo') { // Navigate to demo/ where pom.xml is located
                    bat 'mvn clean install' // Windows
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
            dir('demo') { // Archive artifacts and test results from demo/
                archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
                junit 'target/surefire-reports/*.xml'
            }
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