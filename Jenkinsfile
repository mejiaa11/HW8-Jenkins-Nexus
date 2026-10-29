pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    tools {
        maven '3.9.11'
    }

    environment {
        NEXUS_RELEASE_REPO_ID = 'nexus-releases'
        NEXUS_SNAPSHOT_REPO_ID = 'nexus-snapshots'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASSWORD'
                )]) {
                    writeFile file: 'nexus-settings.xml', text: """<settings>
    <servers>
        <server>
            <id>${NEXUS_RELEASE_REPO_ID}</id>
            <username>${NEXUS_USER}</username>
            <password>${NEXUS_PASSWORD}</password>
        </server>
        <server>
            <id>${NEXUS_SNAPSHOT_REPO_ID}</id>
            <username>${NEXUS_USER}</username>
            <password>${NEXUS_PASSWORD}</password>
        </server>
    </servers>
</settings>"""

                    sh 'mvn deploy -DskipTests -s nexus-settings.xml'
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f nexus-settings.xml'
            echo 'Pipeline finished.'
        }
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}