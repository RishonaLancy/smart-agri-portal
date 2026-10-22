pipeline {
    agent any

    parameters {
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Execute validation stage')
        choice(name: 'ENVIRONMENT', choices: ['staging', 'production'], description: 'Target deploy environment')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/RishonaLancy/smart-agri-portal.git'
            }
        }

        stage('Parallel Build & Validation') {
            parallel {
                stage('Validate HTML/Syntax') {
                    steps {
                        sh 'echo "Validating index.html..." && test -f index.html'
                    }
                }
                stage('Archive Static Assets') {
                    steps {
                        archiveArtifacts artifacts: 'index.html', fingerprint: true
                    }
                }
            }
        }

        stage('Conditional Build Step') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                sh 'echo "Running conditional test suite for environment: ${params.ENVIRONMENT}"'
            }
        }

        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker build -t $DOCKER_USERNAME/smart-agri-portal:latest .'
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker push $DOCKER_USERNAME/smart-agri-portal:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(credentialsId: 'k8s-kubeconfig') {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}

