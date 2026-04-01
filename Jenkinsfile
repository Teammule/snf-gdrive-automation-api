pipeline {
 
    agent any
 
    // ============================================================
    // TOOLS — Force Jenkins to use Java 21 for Mule 4.11.0
    // ============================================================
    tools {
        jdk 'Java21'
    }
 
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
 
        // ── Maven Settings ──
        MAVEN_OPTS             = '-Xmx1024m'
        // Points to Jenkins LocalSystem .m2 settings.xml
        MAVEN_SETTINGS         = 'C:\\Windows\\system32\\config\\systemprofile\\.m2\\settings.xml'
    }
 
    // ============================================================
    // PIPELINE OPTIONS
    // ============================================================
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
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
                echo 'Code checked out successfully!'
            }
        }
 
        // ── STAGE 2: Build ───────────────────────────────────────
        stage('Build') {
            steps {
                echo '========== STAGE 2: Maven Build =========='
                bat "mvn clean package -DskipTests -Dapp.runtime=%MULE_VERSION% -s %MAVEN_SETTINGS%"
                echo 'Build Successful!'
            }
        }
 
        // ── STAGE 3: Run MUnit Tests ─────────────────────────────
        stage('MUnit Tests') {
            steps {
                echo '========== STAGE 3: Running MUnit Tests =========='
                bat "mvn test -Dapp.runtime=%MULE_VERSION% -s %MAVEN_SETTINGS%"
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: '**/target/surefire-reports/*.xml'
                    echo 'MUnit Test Results Published!'
                }
                failure {
                    echo 'MUnit Tests FAILED! Stopping pipeline.'
                }
            }
        }
 
        // ── STAGE 4: Publish to Anypoint Exchange ────────────────
        stage('Publish to Exchange') {
            steps {
                echo '========== STAGE 4: Publish to Anypoint Exchange =========='
                // Publish artifact to Exchange FIRST before CloudHub deploy
                bat """
                    mvn deploy ^
                    -DskipDeploymentVerification ^
                    -Danypoint.client.id=%ANYPOINT_CLIENT_ID% ^
                    -Danypoint.client.secret=%ANYPOINT_CLIENT_SECRET% ^
                    -s %MAVEN_SETTINGS% ^
                    -DskipTests
                """
                echo 'Published to Anypoint Exchange!'
            }
        }
 
        // ── STAGE 5: Deploy to CloudHub 2.0 ─────────────────────
        stage('Deploy to CloudHub 2.0') {
            steps {
                echo '========== STAGE 5: Deploy to CloudHub 2.0 =========='
                echo "Deploying ${CLOUDHUB_APP_NAME} to ${CLOUDHUB_ENVIRONMENT} environment..."
                bat """
                    mvn deploy -DmuleDeploy ^
                    -Danypoint.client.id=%ANYPOINT_CLIENT_ID% ^
                    -Danypoint.client.secret=%ANYPOINT_CLIENT_SECRET% ^
                    -Dcloudhub.application.name=%CLOUDHUB_APP_NAME% ^
                    -Dcloudhub.environment=%CLOUDHUB_ENVIRONMENT% ^
                    -Dcloudhub.businessGroupId=%CLOUDHUB_BG_ID% ^
                    -Dcloudhub.target=%CLOUDHUB_TARGET% ^
                    -Dcloudhub.muleVersion=%MULE_VERSION% ^
                    -s %MAVEN_SETTINGS% ^
                    -DskipTests
                """
                echo 'Deployment to CloudHub 2.0 Successful!'
            }
        }
 
    }
 
    // ============================================================
    // POST PIPELINE ACTIONS
    // ============================================================
    post {
        success {
            echo '============================================'
            echo 'PIPELINE SUCCEEDED!'
            echo "App        : ${env.CLOUDHUB_APP_NAME}"
            echo "Environment: ${env.CLOUDHUB_ENVIRONMENT}"
            echo 'Region     : Cloudhub-US-East-2'
            echo '============================================'
        }
        failure {
            echo '============================================'
            echo 'PIPELINE FAILED!'
            echo 'Please check the logs above for errors.'
            echo '============================================'
        }
        always {
            echo '========== Pipeline Execution Completed =========='
            cleanWs()
        }
    }
 
}