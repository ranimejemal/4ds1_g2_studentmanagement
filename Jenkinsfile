pipeline {
    agent any

    tools {
        maven 'M3'      // adapter aux noms configures dans Manage Jenkins > Tools
        jdk 'jdk17'
    }

    triggers {
        pollSCM('* * * * *')   // inchange : reprenez votre declencheur existant
    }

    environment {
    IMAGE_NAME = 'ranimejemal/student-management'
    IMAGE_TAG  = "${env.BUILD_NUMBER}"
}

    stages {
        // --- Stages existants : reprenez les votres a l'identique ---
        stage('Commit') {
            steps {
                git branch: 'main', url: 'https://github.com/VOTRE_COMPTE/student-management.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B clean package -DskipTests'
            }
        }
        stage('Test unitaire') {
            steps {
                sh 'mvn -B test'
            }
        }

        // --- Nouveaux stages ---
        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG -t $IMAGE_NAME:latest .'
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS')]) {
                    // Apostrophes : le shell, et non Groovy, developpe la variable secrete
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
                    sh 'docker push $IMAGE_NAME:latest'
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
            archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            sh 'docker logout || true'
        }
        success {
            echo "Pipeline OK : image ${IMAGE_NAME}:${IMAGE_TAG} publiee."
        }
        failure {
            echo 'Pipeline en echec : aucune image publiee pour ce build.'
        }
    }
}