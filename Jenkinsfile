// pipeline {
//     agent any
//        options{
//         timeout(time: 10, unit: 'MINUTES')
//         disableConcurrentBuilds()
//         retry(1)
//     }
//     environment {
//         DEBUG = 'true'
//         appVersion = ''
//         region = 'us-east-1'
//         project = 'expense'
//         accountid = '311141538313'
//         component = 'backend'
//         environment = 'dev'
//     }
//     stages {
//         stage('Print the version') {
//             steps {
//                     script{
//                         def packageJson = readJSON file: 'package.json'
//                         appVersion = packageJson.version
//                         echo "App version: ${appVersion}"
//                 }
//             }
//         }
//         stage('Install dependencies') {
//             steps {
//                 sh 'npm install'
//             }
//         }

//         // stage('SonarQube analysis') {
//         //     environment {
//         //         SCANNER_HOME = tool 'sonar-6.0' //scanner config
//         //     }
//         //     steps {
//         //         // sonar server injection
//         //         withSonarQubeEnv('sonar-6.0') {
//         //             sh '$SCANNER_HOME/bin/sonar-scanner'
//         //             //generic scanner, it automatically understands the language and provide scan results
//         //         }
//         //     }
//         // }
//         // here we are pushing docker image to ECR. Taking the PUSH commands from ecr repository and using here
//         stage ('Docker build') {
//             steps {
//                 withAWS(region: 'us-east-1', credentials: 'aws-creds') {
//                 sh """
//                     aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${accountid}.dkr.ecr.us-east-1.amazonaws.com
//                     docker build -t ${accountid}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion} .
//                     docker images
//                     docker push ${accountid}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion}
                    

//                 """
//                 }
//             }
//         }
//         stage ('Deploy') {
//             steps {
//                 withAWS(region: 'us-east-1', credentials: 'aws-creds'){
//                     sh """
//                         aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
//                         cd helm-backend
//                         sed -i 's/IMAGE_VERSION/${appVersion}/g' values.yaml
//                         helm upgrade --install ${component} -n ${project} -f values.yaml .
//                     """
//                 }
//             }
//         }
//     }
// }



pipeline {
    agent any
    options {
        timeout(time: 20, unit: 'MINUTES')
        disableConcurrentBuilds()
        retry(1)
    }
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: 'Select deployment environment')
    }
    environment {
        DEBUG = 'true'
        appVersion = ''
        region = ''
        accountid = ''
        project = 'expense'
        component = 'backend'
        NODE_HOME = tool name: 'nodejs', type: 'NodeJS'  // Automatically use the installed Node.js
    }
    stages {
        stage('Set Environment Variables') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'dev') {
                        region = 'us-east-1'
                        accountid = '311141538313'
                        project = 'expense-dev'
                    } else if (params.ENVIRONMENT == 'test') {
                        region = 'us-west-2'
                        accountid = '311141538314'
                        project = 'expense-test'
                    } else if (params.ENVIRONMENT == 'prod') {
                        region = 'us-west-1'
                        accountid = '311141538315'
                        project = 'expense-prod'
                    }

                    echo "Deploying to ${params.ENVIRONMENT} environment"
                    echo "Region: ${region}, Account ID: ${accountid}, Project: ${project}"
                }
            }
        }
        
        stage('Print the version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "App version: ${appVersion}"
                }
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'  // Now npm should be available if Node.js is installed correctly
            }
        }

        stage('SonarQube analysis') {
            environment {
                SCANNER_HOME = tool 'sonar-6.0'
            }
            steps {
                withSonarQubeEnv('sonar-6.0') {
                    sh '$SCANNER_HOME/bin/sonar-scanner'
                }
            }
        }

        stage('Docker build') {
            steps {
                withAWS(region: "${region}", credentials: 'aws-creds') {
                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${accountid}.dkr.ecr.${region}.amazonaws.com
                        docker build -t ${accountid}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion} .
                        docker images
                        docker push ${accountid}.dkr.ecr.${region}.amazonaws.com/${project}/${component}:${appVersion}
                    """
                }
            }
        }

        stage('Deploy to ${params.ENVIRONMENT}') {
            steps {
                withAWS(region: "${region}", credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-${params.ENVIRONMENT}
                        cd helm-backend
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${params.ENVIRONMENT}.yaml
                        helm upgrade --install ${component} -n ${project} -f values-${params.ENVIRONMENT}.yaml .
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Deployment to ${params.ENVIRONMENT} was successful!"
        }
        failure {
            echo "Deployment to ${params.ENVIRONMENT} failed."
        }
    }
}
