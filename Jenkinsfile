pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Source code checked out successfully"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Local') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package & Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
                echo "Artifact archived successfully"
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {

                    sh '''
                        cat > nexus-settings.xml <<EOF
<settings>
    <servers>
        <server>
            <id>nexus</id>
            <username>${NEXUS_USERNAME}</username>
            <password>${NEXUS_PASSWORD}</password>
        </server>
    </servers>
</settings>
EOF

                        mvn deploy \
                          -s nexus-settings.xml \
                          -DaltDeploymentRepository=nexus::default::http://localhost:8081/repository/techbuild-releases/
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
            echo "Artifact deployed to Nexus successfully!"
        }

        failure {
            echo "Pipeline failed."
        }
    }
}
