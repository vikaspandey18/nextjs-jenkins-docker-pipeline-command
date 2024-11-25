# nextjs-jenkins-docker-pipeline-command

pipeline {
    agent any
    
    options {
        skipDefaultCheckout() // Avoid Jenkins automatic checkout
    }

    stages {
        stage('Pushing Code To Github') {
            steps {
                git branch: 'main', url: 'https://github.com/vikaspandey18/nextjsproject2.git'
            }
        }
        stage("Create a Docker Image"){
            steps{
                sh "docker image build -t vikas181996/nextjsdevops:v$BUILD_ID ."
                sh "docker image tag vikas181996/nextjsdevops:v$BUILD_ID vikas181996/nextjsdevops:latest"
            }
        }
        stage("Pushing Image to Docker Hub"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockercredentials', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                    sh '''
                        echo "${dockerpassword}" | docker login -u "${dockerusername}" --password-stdin
                        docker image push vikas181996/nextjsdevops:v$BUILD_ID
                        docker image push vikas181996/nextjsdevops:latest
                        docker image rmi vikas181996/nextjsdevops:v$BUILD_ID
                        docker image rmi vikas181996/nextjsdevops:latest
                    '''
                }
            }
        }
        stage("Creating Container"){
            steps{
                script {
                    // Stop and remove the existing container if it exists
                    sh '''
                        if [ "$(docker ps -aq -f name=nextjsproject)" ]; then
                            docker stop nextjsproject || true
                            docker rm nextjsproject || true
                        fi
                        docker run -itd --name=nextjsproject -p 3000:3000 vikas181996/nextjsdevops:latest
                    '''
                }
            }
        }
        stage('Cleanup') {
            steps {
                echo "Cleaning up unused Docker resources..."
                sh "docker image prune -f"
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Please check the logs for details."
        }
        always {
            echo "Pipeline execution completed."
        }
    }
}

sudo chmod 666 /var/run/docker.sock
