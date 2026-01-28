pipeline {
    agent any

    environment {
        MICROSERVICE = 'EnfiniteCBS_UI'
        IMAGE_NAME   = 'test-image'
    }

    stages {

        stage('Cleanup Workspace') {
            steps { cleanWs() }
        }

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build') {
            steps {
                sh 'echo "Build step (test only)"'
            }
        }

        stage('Increment Version') {
            steps {
                script {
                    // maan lo current version ye hai
                    def currentVersion = "26.01.00.00.03"
                    echo "Old Version: ${currentVersion}"

                    def parts = currentVersion.split('\\.')
                    def next = parts[4].toInteger() + 1
                    parts[4] = String.format("%02d", next)

                    env.NEW_TAG = parts.join('.')
                    echo "New Version: ${env.NEW_TAG}"
                }
            }
        }

        stage('Docker Build (NO PUSH)') {
            steps {
                sh '''
                  echo "Docker build only"
                  echo "docker build -t ${IMAGE_NAME}:${NEW_TAG} ."
                '''
            }
        }

        stage('Deploy (Dummy)') {
            steps {
                sh '''
                  echo "Deploying version ${NEW_TAG}"
                  echo "Test only – no k8s"
                '''
            }
        }
    }

    post {
        success {
            echo "SUCCESS with version ${NEW_TAG}"
        }
        failure {
            echo "FAILED"
        }
    }
}
