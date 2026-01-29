pipeline {

    agent any
 
    environment {

        MICROSERVICE = 'EnfiniteCBS_UI'

        IMAGE_NAME   = 'test-image'
 
        // PostgreSQL details

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

                withCredentials([usernamePassword(

                    credentialsId: 'postgres-creds',

                    usernameVariable: 'DB_USER',

                    passwordVariable: 'DB_PASS'

                )]) {

                    script {
 
                        echo "Incrementing version from PostgreSQL..."
 
                        def buildNo = sh(

                            script: """

                            PGPASSWORD=$DB_PASS psql \

                              -h 10.150.17.37 \

                              -U $DB_USER \

                              -d versiondb \

                              -t -A \

                              -c "

                              INSERT INTO version_store (service, year, month, build)

                              VALUES ('${MICROSERVICE}', '26', '01', 1)

                              ON CONFLICT (service, year, month)

                              DO UPDATE

                                SET build = version_store.build + 1

                              RETURNING build;

                              "

                            """,

                            returnStdout: true

                        ).trim()
 
                        env.NEW_TAG = "26.01.00.00.${String.format('%02d', buildNo.toInteger())}"
 
                        echo "✅ New Version Generated: ${env.NEW_TAG}"

                    }

                }

            }

        }
 
        stage('Docker Build (NO PUSH)') {

            steps {

                sh """

                  echo "Docker build command:"

                  echo "docker build -t ${IMAGE_NAME}:${NEW_TAG} ."

                """

            }

        }
 
        stage('Deploy (Dummy)') {

            steps {

                sh """

                  echo "Deploying ${IMAGE_NAME}:${NEW_TAG}"

                  echo "Dummy deploy – no k8s yet"

                """

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

 
