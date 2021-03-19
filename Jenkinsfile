pipeline {
    agent { label 'docker' } 
    environment{
        CONTAINER_NAME = "anura"
        PORT="3001"
    }
    stages{
        stage("Build"){
            steps{
                echo "Create the Docker Image"
                sh "docker build -t ${CONTAINER_NAME}-redis:1.0 ."
            }
        }
        stage("Deploy"){
            steps{
                sh "docker run --name ${CONTAINER_NAME} -p ${PORT}:6379 -d --restart unless-stopped ${CONTAINER_NAME}-redis:1.0"
            }
        }
        stage("Test"){
          
            steps{
                echo "Tests"
            }
        }
    }
    post {
       
        always{
            echo "Build number: ${BUILD_NUMBER}"
        }
        success{echo "Success!"}
        failure{echo "failure!"}
    }
}