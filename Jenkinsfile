pipeline {
    agent {
        label 'roboshop-dev'
    }

    environment {
        appVersion = ''
        REGION  = "us-east-1"
        ACC_ID  = "108717859359"
        PROJECT =  "roboshop"
        COMPONENT = "catalogue"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the environment')
        
    }

    stages {
        
    stage('Deploy') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Package version: ${appVersion}"
                }
            }
        }
       
        stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'jenkins-ecr-user', region: 'us-east-1')
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                        """
                }
            }
        }
    }

    post {
        always {
            echo "Say hello"
            deleteDir()
        }
        success {
            echo "pipeline is success"
        }
        failure {
            echo "pipeline is failure"
            }
        }
    }