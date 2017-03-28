#!groovy
pipeline {
    agent any
    environment {
        IBM_CLOUD_DEVOPS_CREDS = credentials('82789767-7033-4ae9-90a0-436e84468f54')
        IBM_CLOUD_DEVOPS_ORG = 'jichen@us.ibm.com'
        IBM_CLOUD_DEVOPS_APP_NAME = 'Weather V1'
        IBM_CLOUD_DEVOPS_TOOLCHAIN_ID = '1f4bdb54-fa86-4e06-9530-474d644547f3'
        CF_API="https://api.ng.bluemix.net"
    }
    tools {
        nodejs 'recent NodeJS'
    }
    stages {
        stage('Build') {
            environment {
                GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                GIT_BRANCH = 'master'
            }
            steps {
                checkout scm

                sh 'npm --version'
                sh 'npm install'
                sh 'grunt dev-setup --no-color'
            }
            post {
                success {
                    publishBuildRecord gitBranch: "${GIT_BRANCH}", gitCommit: "${GIT_COMMIT}", gitRepo: "https://github.com/jinfangchen/weather-InterConnectDemo-0306", result:"SUCCESS"
                }
                failure {
                    publishBuildRecord gitBranch: "${GIT_BRANCH}", gitCommit: "${GIT_COMMIT}", gitRepo: "https://github.com/jinfangchen/weather-InterConnectDemo-0306", result:"FAIL"
                }
            }
        }
        stage('Unit Test and Code Coverage') {
            steps {
                sh 'grunt dev-test-cov --no-color -f'
            }
            post {
                always {
                    publishTestResult type:'unittest', fileLocation: './mochatest.json'
                    publishTestResult type:'code', fileLocation: './tests/coverage/reports/coverage-summary.json'
                }
            }
        }
        stage('Deploy to Staging') {
            environment {
                CF_SPACE='staging'
            }
            steps {
                echo "deploying to staging"
                sh '''
                        echo "CF Login..."
                        cf api $CF_API
                        cf login -u $IBM_CLOUD_DEVOPS_CREDS_USR -p $IBM_CLOUD_DEVOPS_CREDS_PSW -o $IBM_CLOUD_DEVOPS_ORG -s $CF_SPACE
                        echo "Deploying...."
                        export CF_APP_NAME="staging-$IBM_CLOUD_DEVOPS_APP_NAME"
                        cf delete $CF_APP_NAME -f
                        cf push $CF_APP_NAME -n $CF_APP_NAME -m 64M -i 1
                        export APP_URL=http://$(cf app $CF_APP_NAME | grep urls: | awk '{print $2}')
                    '''
            }
            post {
                success {
                    publishDeployRecord environment: "STAGING", appUrl: "http://staging-${IBM_CLOUD_DEVOPS_APP_NAME}.mybluemix.net", result:"SUCCESS"
                }
                failure {
                    publishDeployRecord environment: "STAGING", appUrl: "http://staging-${IBM_CLOUD_DEVOPS_APP_NAME}.mybluemix.net", result:"FAIL"
                }
            }
        }
        stage('FVT') {
            environment {
                APP_URL = "http://staging-${IBM_CLOUD_DEVOPS_APP_NAME}.mybluemix.net"
            }
            steps {
                sh 'grunt fvt-test --no-color -f'
            }
            post {
                always {
                    publishTestResult type:'fvt', fileLocation: './mochafvt.json', environment: 'STAGING'
                }
            }
        }
        stage('Gate') {
            steps {
                evaluateGate policy: ' Weather PRODUCTION Deployment Checks', forceDecision: 'true'
            }
        }
        stage('Deploy to Prod') {
            environment {
                CF_SPACE='prod'
            }
            steps {
                echo "deploying to production"
                sh '''
                        echo "CF Login..."
                        cf api $CF_API
                        cf login -u $IBM_CLOUD_DEVOPS_CREDS_USR -p $IBM_CLOUD_DEVOPS_CREDS_PSW -o $IBM_CLOUD_DEVOPS_ORG -s $CF_SPACE
                        echo "Deploying...."
                        export CF_APP_NAME="prod-$IBM_CLOUD_DEVOPS_APP_NAME"
                        cf delete $CF_APP_NAME -f
                        cf push $CF_APP_NAME -n $CF_APP_NAME -m 64M -i 1
                        export APP_URL=http://$(cf app $CF_APP_NAME | grep urls: | awk '{print $2}')
                    '''
            }
            post {
                success {
                    publishDeployRecord environment: "PRODUCTION", appUrl: "http://prod-${IBM_CLOUD_DEVOPS_APP_NAME}.mybluemix.net", result:"SUCCESS"
                }
                failure {
                    publishDeployRecord environment: "PRODUCTION", appUrl: "http://prod-${IBM_CLOUD_DEVOPS_APP_NAME}.mybluemix.net", result:"FAIL"
                }
            }
        }
 }
