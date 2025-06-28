pipeline {
    agent any

    environment {
        SONARQUBE = 'SonarQubeServer' // Configure this in Jenkins > Manage Jenkins > Tools
        GMAIL_USER = credentials('gmail-user')  // Add credentials in Jenkins
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/yourusername/yourrepo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test -- --coverage' // Jest
                junit 'test-results.xml'
            }
        }

        stage('Lint Code') {
            steps {
                sh 'npx eslint .'
            }
        }

        stage('Code Coverage Check') {
            steps {
                script {
                    def coverage = readFile('coverage/coverage-summary.json')
                    def json = new groovy.json.JsonSlurper().parseText(coverage)
                    if (json.total.statements.pct < 80) {
                        error("Code coverage too low!")
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'npx sonar-scanner'
                }
            }
        }

        stage('Archive Artifacts') {
            when {
                expression { return env.GIT_TAG_NAME?.contains("release") }
            }
            steps {
                archiveArtifacts artifacts: '**/*.zip', fingerprint: true
            }
        }
    }

    post {
        success {
            emailext (
                subject: "✅ Build Successful",
                body: "Build passed for ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                to: "recipient@gmail.com"
            )
        }
        failure {
            emailext (
                subject: "❌ Build Failed",
                body: "Build failed for ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                to: "recipient@gmail.com"
            )
        }
    }
}
