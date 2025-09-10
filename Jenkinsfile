pipeline {
    agent any

    tools {
        maven "MAVEN_HOME"
    }

    stages {
        stage('Git Clone') {
            steps {
                git url: 'https://github.com/Abdulaziz920/hiring-app.git', branch: 'main'
            }
        }

        stage('Maven Package') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar, target/*.war', fingerprint: true
            }
        }
    }
}
