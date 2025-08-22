pipeline {
    agent  {
        label 'AGENT-1'
    }
    environment { 
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "108717859359"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }

    stages {
        stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'jenkins-ecr-user', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }

       stage('Check status') {
    steps {
        script {
            withAWS(credentials: 'jenkins-ecr-user', region: 'us-east-1') {
                def deploymentStatus = sh(returnStdout: true, script: """
                    kubectl rollout status deployment/${COMPONENT} --timeout=120s -n $PROJECT || echo FAILED
                """).trim()

                if (deploymentStatus.contains("successfully rolled out")) {
                    echo "Deployment is success"
                } else {
                    echo "Deployment failed. Attempting rollback..."
                    // Get previous Helm revision
                    def prevRevision = sh(returnStdout: true, script: """
                        helm history $COMPONENT -n $PROJECT --max=2 --output json | jq -r '.[-2].revision // empty'
                    """).trim()

                    if (prevRevision) {
                        echo "Rolling back to revision ${prevRevision}"
                        sh "helm rollback $COMPONENT $prevRevision -n $PROJECT"
                        sleep 20
                        def rollbackStatus = sh(returnStdout: true, script: """
                            kubectl rollout status deployment/${COMPONENT} --timeout=120s -n $PROJECT || echo FAILED
                        """).trim()

                        if (rollbackStatus.contains("successfully rolled out")) {
                            error "Deployment failed, rollback succeeded"
                        } else {
                            error "Deployment failed, rollback also failed. Application is not running"
                        }
                    } else {
                        echo "No previous revision found. Cannot rollback."
                        error "Deployment failed and no rollback possible"
                    }
                }
            }
        }
    }
}


    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'Hello Success'
        }
        failure { 
            echo 'Hello Failure'
        }
    }
}
