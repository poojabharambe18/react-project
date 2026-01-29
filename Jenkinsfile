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

                        # Store SQL in a variable to avoid quoting issues

                        SQL_COMMAND="

                            INSERT INTO version_store (service, year, month, build)

                            VALUES ('EnfiniteCBS_UI', '26', '01', 1)

                            ON CONFLICT (service, year, month)

                            DO UPDATE SET build = version_store.build + 1

                            RETURNING build;

                        "

                        psql -t -A -c "$SQL_COMMAND"

                    '''

                ).trim()
 
                env.NEW_TAG = "26.01.00.00.${String.format('%02d', buildNo.toInteger())}"

                echo "✅ New Version Generated: ${env.NEW_TAG}"

            }

        }

    }

}
 
