pipeline {
    agent {
        label 'AGENT-1'
    }
       options{
        timeout(time: 10, unit: 'SECONDS')
        disableConcurrentBuilds()
        retry(1)
    }
    environment {
        DEBUG = 'true'
        appVersion = ''
    }
    stages {
        stage('Print the version') {
            steps {
                    script{
                        def packageJson = readJSON file: 'package.json'
                        appVersion = packageJson.version
                        echo "App version: ${appVersion}"
                }
            }
        }
        stage('Test') {
              options {
                 timeout(time: 10, unit: 'SECONDS')
             }
             steps {
                sh 'echo   This is Test'
                // sh 'sleep 11'
            }
        }
        stage('Deploy') {
            when {
                expression { env.GIT_BRANCH == "origin/main" }
            }
            steps {
                sh 'echo This is  deploy'
            }
        }
    }
}