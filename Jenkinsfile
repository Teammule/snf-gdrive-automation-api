pipeline {
 
agent any
 
tools {
jdk 'Java21'
}
 
environment {
 
// Connected App Credentials (Jenkins Secrets)
ANYPOINT_CLIENT_ID = credentials('anypoint-client-id')
ANYPOINT_CLIENT_SECRET = credentials('anypoint-client-secret')
 
// CloudHub 2.0 Deployment Config
CLOUDHUB_APP_NAME = 'snowflake-gdrive-dev'
CLOUDHUB_ENVIRONMENT = 'Sandbox'
CLOUDHUB_BG_ID = '7603b4c1-08c5-4c45-b0a0-1c3c06cad292'
CLOUDHUB_TARGET = 'Cloudhub-US-East-2'
MULE_VERSION = '4.11.0'
 
MAVEN_SETTINGS = 'C:\\Windows\\system32\\config\\systemprofile\\.m2\\settings.xml'
}
 
options {
buildDiscarder(logRotator(numToKeepStr: '5'))
timeout(time: 30, unit: 'MINUTES')
disableConcurrentBuilds()
timestamps()
}
 
stages {
 
stage('Checkout') {
steps {
echo '===== CHECKOUT SOURCE ====='
checkout scm
}
}
 
stage('Build Artifact') {
            steps {
                echo '===== COMPILING MULE APPLICATION ====='
                // Added -U to force Maven to clear its cache and download the new MUnit files
                bat "mvn clean package -DskipTests -U -s %MAVEN_SETTINGS%"
            }
        }

        stage('MUnit Tests') {
            steps {
                echo '===== RUNNING MUNIT TESTS & CHECKING COVERAGE ====='
                // Added -U here as well just to be safe!
                bat "mvn clean test -U -s %MAVEN_SETTINGS%"
            }
        }
 
stage('Publish to Exchange') {
            // ONLY run this stage if the branch is 'master'
            when {
                expression { env.GIT_BRANCH == 'origin/master' || env.GIT_BRANCH == 'master' }
            }
            steps {
                echo '===== STAMPING NEW VERSION INTO POM.XML ====='
                // This physically changes <version>1.0.0</version> to <version>1.0.49</version>
                bat "mvn versions:set -DnewVersion=1.0.%BUILD_NUMBER% -s %MAVEN_SETTINGS%"
             
                echo '===== PUBLISHING ARTIFACT TO EXCHANGE ====='
                bat """
                mvn clean deploy ^
                -DskipTests ^
                -Danypoint.client.id=%ANYPOINT_CLIENT_ID% ^
                -Danypoint.client.secret=%ANYPOINT_CLIENT_SECRET% ^
                -s %MAVEN_SETTINGS%
                """
            }
        }

        stage('Deploy to CloudHub 2.0') {
            // ONLY run this stage if the branch is 'master'
            when {
                expression { env.GIT_BRANCH == 'origin/master' || env.GIT_BRANCH == 'master' }
            }
            steps {
                echo '===== DEPLOYING TO CLOUDHUB 2.0 (US EAST OHIO) ====='
                bat """
                mvn mule:deploy ^
                -Danypoint.client.id=%ANYPOINT_CLIENT_ID% ^
                -Danypoint.client.secret=%ANYPOINT_CLIENT_SECRET% ^
                -Dcloudhub.application.name=%CLOUDHUB_APP_NAME% ^
                -Dcloudhub.environment=%CLOUDHUB_ENVIRONMENT% ^
                -Dcloudhub.businessGroupId=%CLOUDHUB_BG_ID% ^
                -Dcloudhub.target=%CLOUDHUB_TARGET% ^
                -Dcloudhub.muleVersion=%MULE_VERSION% ^
                -DskipTests ^
                -s %MAVEN_SETTINGS%
                """
            }
        }
}
post {
 
success {
echo '''
============================================
PIPELINE SUCCESSFUL
Application : snowflake-gdrive-dev
Environment : DEV
Region : US East (Ohio)
============================================
'''
}
 
failure {
echo '''
============================================
PIPELINE FAILED
Check console logs for root cause
============================================
'''
}
 
always {
cleanWs()
}
}
}
