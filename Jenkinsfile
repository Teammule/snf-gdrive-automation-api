pipeline {

    agent any

    // ============================================================
    // ENVIRONMENT VARIABLES
    // ============================================================
    environment {

        // ── Anypoint Connected App Credentials (from Jenkins) ──
        ANYPOINT_CLIENT_ID     = credentials('anypoint-client-id')
        ANYPOINT_CLIENT_SECRET = credentials('anypoint-client-secret')

        // ── CloudHub 2.0 Deployment Config ──
        CLOUDHUB_APP_NAME      = 'snowflake-gdrive-dev'
        CLOUDHUB_ENVIRONMENT   = 'DEV'
        CLOUDHUB_BG_ID         = '7603b4c1-08c5-4c45-b0a0-1c3c06cad292'
        CLOUDHUB_TARGET        = 'Cloudhub-US-East-2'
        MULE_VERSION           = '4.11.0'

        // ── Maven ──
        MAVEN_OPTS             = '-Xmx1024m'
    }

    // ============================================================
    // PIPELINE OPTIONS
    // ============================================================
    options {
        // Keep last 5 build logs only
        buildDiscarder(logRotator(numToKeepStr: '5'))

        // Fail if pipeline runs more than 30 minutes
        timeout(time: 30, unit: 'MINUTES')

        // Don't run concurrent builds
        disableConcurrentBuilds()

        // Add timestamps to console logs
        timestamps()
    }

    // ============================================================
    // STAGES
    // ============================================================
    stages {

        // ── STAGE 1: Checkout Code from GitHub ──────────────────
        stage('Checkout') {
            steps {
                echo '========== STAGE 1: Checkout Code =========='
                checkout scm
                echo "Code checked out from branch: ${env.BRANCH_NAME}"
            }
        }

        // ── STAGE 2: Build ───────────────────────────────────────
        stage('Build') {
            steps {
                echo '========== STAGE 2: Maven Build =========='
                sh '''
                    mvn clean package \
                        -DskipTests \
                        -Dapp.runtime=${MULE_VERSION}
                '''
                echo 'Build Successful!'
            }
        }

        // ── STAGE 3: Run MUnit Tests ─────────────────────────────
        stage('MUnit Tests') {
            steps {
                echo '========== STAGE 3: Running MUnit Tests =========='
                sh '''
                    mvn test \
                        -Dapp.runtime=${MULE_VERSION}
                '''
            }
            post {
                always {
                    // Publish MUnit test results in Jenkins
                    junit allowEmptyResults: true,
                          testResults: '**/target/surefire-reports/*.xml'
                    echo 'MUnit Test Results Published!'
                }
                failure {
                    echo 'MUnit Tests FAILED! Stopping pipeline.'
                }
            }
        }

        // ── STAGE 4: Deploy to CloudHub 2.0 ─────────────────────
        stage('Deploy to CloudHub 2.0') {
            steps {
                echo '========== STAGE 4: Deploy to CloudHub 2.0 =========='
                echo "Deploying ${CLOUDHUB_APP_NAME} to ${CLOUDHUB_ENVIRONMENT} environment..."
                sh '''
                    mvn deploy -DmuleDeploy \
                        -Danypoint.client.id=${ANYPOINT_CLIENT_ID} \
                        -Danypoint.client.secret=${ANYPOINT_CLIENT_SECRET} \
                        -Dcloudhub.application.name=${CLOUDHUB_APP_NAME} \
                        -Dcloudhub.environment=${CLOUDHUB_ENVIRONMENT} \
                        -Dcloudhub.businessGroupId=${CLOUDHUB_BG_ID} \
                        -Dcloudhub.target=${CLOUDHUB_TARGET} \
                        -Dcloudhub.muleVersion=${MULE_VERSION} \
                        -DskipTests
                '''
                echo 'Deployment to CloudHub 2.0 Successful!'
            }
        }

    }

    // ============================================================
    // POST PIPELINE ACTIONS
    // ============================================================
    post {

        success {
            echo '''
            ============================================
            PIPELINE SUCCEEDED!
            App        : snowflake-gdrive-dev
            Environment: DEV
            Region     : Cloudhub-US-East-2
            ============================================
            '''
        }

        failure {
            echo '''
            ============================================
            PIPELINE FAILED!
            Please check the logs above for errors.
            ============================================
            '''
        }

        always {
            echo '========== Pipeline Execution Completed =========='

            // Clean workspace after build
            cleanWs()
        }
    }

}
