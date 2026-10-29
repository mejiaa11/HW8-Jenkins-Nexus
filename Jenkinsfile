
pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    tools {
        maven '3.9.11'
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
                    sh '''
                        set +x
                        umask 077
                        cat > nexus-settings.xml <<EOF
<settings>
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>$NEXUS_USER</username>
            <password>$NEXUS_PASSWORD</password>
        </server>
        <server>
            <id>nexus-snapshots</id>
            <username>$NEXUS_USER</username>
            <password>$NEXUS_PASSWORD</password>
        </server>
    </servers>
</settings>
EOF
                        mvn deploy -DskipTests -s nexus-settings.xml
                    '''
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
