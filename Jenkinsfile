pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'dineshpundkar'
        APP_NAME = 'client_api'
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
        KUBECONFIG = credentials('kubeconfig')
        API_URL = 'https://clients.api.deltacapita.com'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                dir('client_api') {
                    script {
                        def dockerImage = docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}")
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    sh """
                        echo ${DOCKER_CREDENTIALS_PSW} | docker login -u ${DOCKER_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    sh """
                        sed -i 's|image: ${DOCKER_REGISTRY}/${APP_NAME}:.*|image: ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}|' manifests/client_api.yaml
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        export KUBECONFIG=\${KUBECONFIG}
                        
                        #Deploy Nignix Ingress Controller
                        kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
                        
                        # Apply cert-manager
                        kubectl apply -f manifests/cert-manager.yaml
                        
                        # Wait for cert-manager to be ready
                        kubectl wait --for=condition=ready pod -l app.kubernetes.io/instance=cert-manager -n cert-manager --timeout=300s
                        
                        # Apply cluster issuer
                        kubectl apply -f manifests/cluster-issuer.yaml
                        
                        # Apply certificate
                        kubectl apply -f manifests/certificate.yaml
                                                
                        # Apply MongoDB 
                        kubectl apply -f manifests/mongodb.yaml
                        
                        # Deploy the API application
                        kubectl apply -f manifests/client_api.yaml
                        
                        # Apply ingress last
                        kubectl apply -f manifests/ingress.yaml
                        
                        # Wait for the deployment to be ready
                        kubectl wait --for=condition=available deployment/dsp-api --timeout=300s
                    """
                }
            }
        }

        stage('Build Acceptance Test') {
            steps {
                script {
                    // Make the execute_bat.sh script executable
                    sh 'chmod +x ./scripts/execute_bat.sh'
                    
                    // Execute BAT tests and store results
                    def testResult = sh(
                        script: "./scripts/execute_bat.sh ${API_URL}",
                        returnStdout: true
                    ).trim()
                    
                    // Print test results
                    echo "BAT Test Results:"
                    echo testResult
                    
                    // Fail the build if tests failed
                    if (testResult.contains("FAILED")) {
                        error "Build Acceptance Tests failed!"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded! Application deployed successfully.'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
        always {
            sh 'docker logout'
        }
    }
}
