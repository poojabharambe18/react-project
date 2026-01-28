stage('Generate Version from PostgreSQL') {
    steps {
        script {
            def buildNo = sh(
                script: """
                psql -U postgres -d versiondb -t -A -c "
                INSERT INTO version_store (service, year, month, build)
                VALUES ('test-image', '26', '01', 1)
                ON CONFLICT (service, year, month)
                DO UPDATE SET build = version_store.build + 1
                RETURNING build;
                "
                """,
                returnStdout: true
            ).trim()

            env.NEW_TAG = "26.01.00.00.${String.format('%02d', buildNo.toInteger())}"

            echo "Generated Version: ${env.NEW_TAG}"
        }
    }
}
