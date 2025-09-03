pipeline {
    agent any

    environment {
        NEXUS_CRED = 'nexus-creds'
        TOMCAT_CRED = 'tomcat-credentials'
        MVN_TOOL = 'MVN_HOME'         // Maven tool configured in Jenkins
        SONAR_SERVER = 'SonarQube'    // SonarQube server in Jenkins
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/Abdulaziz920/Abidd.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                tool name: "${MVN_TOOL}", type: 'maven'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
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
                        WAR_FILE=$(ls target/*.war | head -n 1)
                        curl -u $TOMCAT_USER:$TOMCAT_PASS \
                             -T $WAR_FILE \
                             http://54.145.142.96:8080/manager/text/deploy?path=/hiring&update=true
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build . -t sabair0509/hiring-app:$BUILD_NUMBER"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub', variable: 'hubPwd')]) {
                    sh "docker login -u sabair0509 -p ${hubPwd}"
                    sh "docker push sabair0509/hiring-app:$BUILD_NUMBER"
                }
            }
        }

        stage('Checkout K8S manifest SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/betawins/Hiring-app-argocd.git'
            }
        }

        stage('Update K8S manifest & push to Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Github_server', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) { 
                        sh '''
                            sed -i "s/5/${BUILD_NUMBER}/g" /var/lib/jenkins/workspace/$JOB_NAME/dev/deployment.yaml
                            git add .
                            git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                            git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/betawins/Hiring-app-argocd.git main
                        '''
                    }
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#jenkins-integration',
                    color: 'good',
                    message: "Hi Team, Jenkins pipeline for jenkins-04 task 2 *SIMPLE CUSTOMER APP* has finished successfully! :white_check_mark:\nDeployed by: Abdul Aziz"
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
                message: ":warning: Jenkins pipeline for *hiring-app* failed! Please check."
            )
        }
    }
}


