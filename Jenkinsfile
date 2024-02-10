#!/usr/bin/env groovy

pipeline {
    agent any
    stages {
        stage('Build and Test') {
            steps {
                checkout scm
                sh "chmod +x gradlew"
                sh "./gradlew clean --no-daemon"
                sh "./gradlew checkstyleNohttp --no-daemon"
                sh "./gradlew npm_install -PnodeInstall --no-daemon"
                sh """
                    curl -Lo ./snyk $(curl -s https://api.github.com/repos/snyk/snyk/releases/latest | grep "browser_download_url.*snyk-linux" | cut -d ':' -f 2,3 | tr -d \" | tr -d ' ')
                    chmod +x snyk
                """
                sh './snyk test --all-projects'
                sh './snyk monitor --all-projects'
                sh "./gradlew test integrationTest -PnodeInstall --no-daemon"
                junit '**/build/**/TEST-*.xml'
                sh "./gradlew npm_run_test -PnodeInstall --no-daemon"
                junit '**/build/test-results/TESTS-*.xml'
                withSonarQubeEnv('sonar') {
                sh "./gradlew sonarqube --no-daemon"
                }
            }
        }

        stage('Deliver') {
            steps {
                script {
                    checkout scm
                    sh "chmod +x gradlew"
                    sh "./gradlew bootJar -x test -Pprod -PnodeInstall --no-daemon"
                    archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            agent any
            steps {
                checkout scm
                sh "az login --service-principal --username \$AZURE_CLIENT_ID --password \$AZURE_CLIENT_SECRET --tenant \$AZURE_TENANT_ID"
                unstash 'jar'
                sh "az webapp deploy --resource-group \"\$AZURE_RESOURCE_GROUP\" --name \"\$AZURE_APP_NAME\" --src-path $(find . -name \"proybaseanggrd2*.jar\") --type jar --verbose"
                sh 'az logout'
            }
        }
    }

    when {
        branch 'feature/jorge'
    }
}