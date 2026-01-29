pipeline {
    agent any

    environment {
        MICROSERVICE = 'EnfiniteCBS_UI'
        IMAGE_NAME   = 'test-image'
        DB_HOST = '10.150.17.37'
        DB_NAME = 'versiondb'
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
                sh 'echo "Build step – placeholder"'
            }
        }

        stage('Increment Version from PostgreSQL') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'postgres-creds',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PASS'
                    )
                ]) {
                    script {
                        def buildNo = sh(
                            returnStdout: true,
                            script: '''
                                export PGHOST="10.150.17.37"
                                export PGDATABASE="versiondb"
                                export PGUSER="$DB_USER"
                                export PGPASSWORD="$DB_PASS"

                                psql -t -A <<EOF
                                    INSERT INTO version_store (service, year, month, build)
                                    VALUES ('EnfiniteCBS_UI', '26', '01', 1)
                                    ON CONFLICT (service, year, month)
                                    DO UPDATE SET build = version_store.build + 1
                                    RETURNING build;
                                EOF
                            '''
                        ).trim()

                        env.NEW_TAG = "26.01.00.00.${String.format('%02d', buildNo.toInteger())}"
                        echo "✅ New Version Generated: ${env.NEW_TAG}"
                    }
                }
            }
        }

        stage('Docker Build (NO PUSH)') {
            steps {
                sh '''
                    echo "Docker build command:"
                    echo "docker build -t ${IMAGE_NAME}:${NEW_TAG} ."
                '''
            }
        }

        stage('Deploy (Dummy)') {
            steps {
                sh '''
                    echo "Deploying ${IMAGE_NAME}:${NEW_TAG}"
                    echo "Dummy deploy – no k8s yet"
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 PIPELINE SUCCESS – Version ${NEW_TAG}"
        }
        failure {
            echo "❌ PIPELINE FAILED"
        }
    }
}
