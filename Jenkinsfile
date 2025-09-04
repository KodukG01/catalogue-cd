pipeline {
    agent {
        label 'roboshop-dev'
    }

    environment {
        REGION    = "us-east-1"
        ACC_ID    = "108717859359"
        PROJECT   = "roboshop"
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
                    withAWS(credentials: 'jenkins-ecr-user', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }

        stage('Check Status') {
            steps {
                script {
                    withAWS(credentials: 'jenkins-ecr-user', region: 'us-east-1') {
                    def deploymentStatus = sh(
                        returnStdout: true,
                        script: "kubectl rollout status deployment/catalogue --request-timeout=30s || echo FAILED"
                    ).trim()

                    if (deploymentStatus.contains("successfully rolled out")) {
                        echo "Deployment is success"
                    } else {
                        sh """
                            helm rollback $COMPONENT -n $PROJECT
                            sleep 20
                        """

                        def rollbackStatus = sh(
                            returnStdout: true,
                            script: "kubectl rollout status deployment/catalogue --request-timeout=30s || echo FAILED"
                        ).trim()

                        if (rollbackStatus.contains("successfully rolled out")) {
                            error "Deployment failed, rollback was successful"
                        } else {
                            error "Deployment failed, rollback also failed. Application not running."
                        }
                    }
                }
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
