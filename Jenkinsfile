pipeline {
    agent any
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
    }
    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: "*/${BRANCH_NAME}"]],
                          userRemoteConfigs: [[url: 'https://github.com/Aaikyam/aaikyam-monitoring-service.git', credentialsId: 'aaikyam-labs']]
                ])
            }
        }
        stage('Deploy') {
            steps {
                script {
                    try {
                        // Deploy the stack
                        sh '''
                            docker stack deploy -c ${DOCKER_COMPOSE_FILE} aaikyam_monitoring_service --with-registry-auth
                            
                            # Wait for deployment to stabilize
                            sleep 10
                            
                            # Check deployment status
                            docker service ls
                        '''
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Deployment failed: ${e.message}"
                    }
                }
            }
        }
    }
    post {
        success {
            emailext subject: "Jenkins Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "The build succeeded!\n\nJob: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\n\nCheck it out at ${env.BUILD_URL}",
                     to: 'admin@aaikyam.in',
                     from: 'jenkins@aaikyam.in'
        }
        failure {
            emailext subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "The build failed.\n\nJob: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\n\nCheck it out at ${env.BUILD_URL}",
                     to: 'admin@aaikyam.in',
                     from: 'jenkins@aaikyam.in'
        }
    }
}
