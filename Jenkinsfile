pipeline {
    agent any
    environment {
        IMAGE = "docker.io/psychovivek/srinivas-argoapp1"
        TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage ("Git Pull") {
            steps {
                git branch: 'main', credentialsId: 'github_pat', poll: false, url: 'https://github.com/ayanchasrinivas/CICD-with-Jenkins.git'
            }
        }
 
        stage ("Build Backend Container Image") {
            steps {
                sh "docker build -t '$IMAGE:$TAG' -t '$IMAGE:latest' -f python/Dockerfile"
            }
        }

        stage ("Push Backend Container Image to Dockerhub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dckr-global-password', passwordVariable: 'DOCKERHUB_PWD', usernameVariable: "DOCKERHUB_USER")]) {
                    sh "echo '$DOCKERHUB_PWD' | docker login -u '$DOCKERHUB_USER' --password-stdin"
                    sh "docker push '$IMAGE:$TAG'"
                    sh "docker push '$IMAGE:latest'"
                }
            }
        }

        stage ("Deploy the Container Image to Server") {
            steps {
                sh "docker pull '$IMAGE:$TAG'"
                sh "docker run -d --name srinivas-flask -p 5000:5000 '$IMAGE:$TAG'"

                sh '''
                cat > deploy-info.txt << EOF
                Build: $BUILD_NUMBER
                Commit: ${GIT_COMMIT}
                Image: $IMAGE:$TAG
                Branch: $GIT_BRANCH
                url: $BUILD_URL
                '''
            archiveArtifacts artifacts: "deploy-info.txt", fingerprint: true
            }
        }

        stage ("Smoke Test the application") {
            steps {
                sh "sleep 2"
                sh "echo 'Hit http://localhost:5000 to see the app.'"
            }
        }

        stage ("Clean Up the workspace") {
            steps {
                cleanWs()
            }
        }
    }
    post {
        success {
            echo "Success"
        }
        failure {
            echo "failure"
        }
        always {
            echo "PIPELINE FINISHED"
        }
    }
}