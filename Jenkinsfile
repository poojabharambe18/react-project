pipeline {
    agent any

    environment {
        SERVICE_NAME = 'test-image'
        YEAR  = '26'
        MONTH = '01'
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build (Dummy)') {
            steps {
                sh 'echo "Build step – test only"'
            }
        }

        stage('Generate Version from PostgreSQL') {
            steps {
                script {

                    def buildNo = sh(
                        script: """
                        sudo psql -U postgres -d versiondb -t -A -c "
                        INSERT INTO version_store (service, year, month, build)
                        VALUES ('${SERVICE_NAME}', '${YEAR}', '${MONTH}', 1)
                        ON CONFLICT (service, year, month)
                        DO UPDATE SET build = version_store.build + 1
                        RETURNING build;
                        "
                        """,
                        returnStdout: true
                    ).trim()

                    env.NEW_TAG = "${YEAR}.${MONTH}.00.00.${String.format('%02d', buildNo.toInteger())}"

                    echo "Generated Version: ${env.NEW_TAG}"
                }
            }
        }

        stage('Docker Build (NO PUSH)') {
            steps {
                sh '''
                    echo "Docker build only"
                    echo "docker build -t test-image:${NEW_TAG} ."
                '''
            }
        }

        stage('Deploy (Dummy)') {
            steps {
                sh '''
                    echo "Deploying version ${NEW_TAG}"
                    echo "Test deployment – no Kubernetes"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ PIPELINE SUCCESS – Version: ${NEW_TAG}"
        }
        failure {
            echo "❌ PIPELINE FAILED"
        }
    }
}
