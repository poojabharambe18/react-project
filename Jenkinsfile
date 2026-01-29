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
                        echo "=== DEBUG: Checking database state ==="

                        sh '''
                            export PGHOST="10.150.17.37"
                            export PGDATABASE="versiondb"
                            export PGUSER="$DB_USER"
                            export PGPASSWORD="$DB_PASS"

                            echo "=== Full table contents ==="
                            psql -c "SELECT * FROM version_store;"

                            echo "=== Query for our service ==="
                            psql -c "SELECT * FROM version_store WHERE service='EnfiniteCBS_UI' AND year='26' AND month='01';"

                            echo "=== COALESCE test ==="
                            psql -t -A -c "SELECT COALESCE(build, 0) FROM version_store WHERE service='EnfiniteCBS_UI' AND year='26' AND month='01';"
                        '''

                        def currentBuild = sh(
                            returnStdout: true,
                            script: '''
                                export PGHOST="10.150.17.37"
                                export PGDATABASE="versiondb"
                                export PGUSER="$DB_USER"
                                export PGPASSWORD="$DB_PASS"
                                psql -t -A -c "
                                    SELECT COALESCE(
                                        (SELECT build FROM version_store
                                         WHERE service='EnfiniteCBS_UI'
                                         AND year='26'
                                         AND month='01'),
                                        0
                                    );
                                "
                            '''
                        ).trim()

                        echo "Current build number raw: '${currentBuild}'"

                        int currentBuildInt
                        try {
                            currentBuildInt = currentBuild.toInteger()
                        } catch (Exception e) {
                            echo "Warning: Could not parse '${currentBuild}' as integer, defaulting to 0"
                            currentBuildInt = 0
                        }

                        echo "Current build number: ${currentBuildInt}"

                        int newBuild = currentBuildInt + 1
                        echo "New build number will be: ${newBuild}"

                        def updatedBuild = sh(
                            returnStdout: true,
                            script: """
                                export PGHOST="10.150.17.37"
                                export PGDATABASE="versiondb"
                                export PGUSER="\$DB_USER"
                                export PGPASSWORD="\$DB_PASS"
                                psql -t -A -c "
                                    INSERT INTO version_store (service, year, month, build)
                                    VALUES ('EnfiniteCBS_UI', '26', '01', ${newBuild})
                                    ON CONFLICT (service, year, month)
                                    DO UPDATE SET build = EXCLUDED.build
                                    RETURNING build;
                                "
                            """
                        ).trim()

                        echo "Updated build number returned: '${updatedBuild}'"

                        sh '''
                            export PGHOST="10.150.17.37"
                            export PGDATABASE="versiondb"
                            export PGUSER="$DB_USER"
                            export PGPASSWORD="$DB_PASS"
                            echo "=== Verification ==="
                            psql -c "SELECT * FROM version_store WHERE service='EnfiniteCBS_UI' AND year='26' AND month='01';"
                        '''

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
