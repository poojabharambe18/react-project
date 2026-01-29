pipeline {
    agent any

    environment {
        MICROSERVICE = 'EnfiniteCBS_UI'
        IMAGE_NAME   = 'test-image'
        DB_HOST      = '10.150.17.37'
        DB_NAME      = 'versiondb'
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
                        def currentBuild = sh(
                            returnStdout: true,
                            script: '''
                                export PGHOST="10.150.17.37"
                                export PGDATABASE="versiondb"
                                export PGUSER="$DB_USER"
                                export PGPASSWORD="$DB_PASS"
                                SQL="SELECT COALESCE(build, 0)
                                     FROM version_store
                                     WHERE service='EnfiniteCBS_UI'
                                     AND year='26'
                                     AND month='01';"
                                psql -t -A -c "$SQL"
                            '''
                        ).trim()

                        echo "Current build number from DB: '${currentBuild}'"

                        if (!currentBuild) {
                            currentBuild = "0"
                        }

                        def newBuild = currentBuild.toInteger() + 1

                        def updatedBuild = sh(
                            returnStdout: true,
                            script: """
                                export PGHOST="10.150.17.37"
                                export PGDATABASE="versiondb"
                                export PGUSER="\${DB_USER}"
                                export PGPASSWORD="\${DB_PASS}"
                                SQL="INSERT INTO version_store (service, year, month, build)
                                     VALUES ('EnfiniteCBS_UI', '26', '01', ${newBuild})
                                     ON CONFLICT (service, year, month)
                                     DO UPDATE SET build=${newBuild}
                                     RETURNING build;"
                                psql -t -A -c "\$SQL"
                            """
                        ).trim()

                        echo "Updated build number in DB: '${updatedBuild}'"

                        env.NEW_TAG = "26.01.00.00.${String.format('%02d', newBuild)}"
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
