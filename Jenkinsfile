pipeline {
    agent any

    environment {
        NEXUS_CRED = 'Nexux_server'
        TOMCAT_CRED = 'Tomcat-credentials'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/Abdulaziz920/hiring-app.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                tool name: 'Maven-3.9.11', type: 'MVN_HOME'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                    mvn deploy -DskipTests \
                        -Dnexus.username=$NEXUS_USER \
                        -Dnexus.password=$NEXUS_PASS \
                        --settings /var/lib/jenkins/.m2/settings.xml
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${TOMCAT_CRED}", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                    # Pick the latest WAR
                    WAR_FILE=$(ls target/*.war | head -n 1)
                    WAR_NAME=$(basename $WAR_FILE .war | tr '[:upper:]' '[:lower:]')

                    echo "Deploying $WAR_FILE to Tomcat at context path /$WAR_NAME..."
                    
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                         -T $WAR_FILE \
                         "http://34.224.101.158:8080/manager/text/deploy?path=/$WAR_NAME&update=true"
                    '''
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#jenkins-integration',
                    color: 'good',
                    message: "Hi Team, Jenkins pipeline for *Declarative pipeline* has finished successfully! ✅\nDeployed by: Abdul Aziz"
                )
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }
        failure {
            slackSend(
                channel: '#jenkins-integration',
                color: 'danger',
                message: "⚠️ Jenkins pipeline for *Simple Customer App* failed! Please check."
            )
        }
    }
}


