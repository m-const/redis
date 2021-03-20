pipeline {
    
    parameters{
        string(name: 'CONTAINER_NAME', defaultValue: 'anura', description: "Docker Container Name")
        //choice(name: 'NAME', choices['3','2','1'], description: "container name")
        booleanParam(name: 'RUNTESTS', defaultValue: false, description: "Run Test Section?")
        booleanParam(name: 'CLEAR_DOCKER', defaultValue: false, description: "Force delete other containers running on this port?")
    }
    environment{
       PORT="3001"
    }
    agent { label 'docker' } 
    stages{
        stage("Build"){
            steps{
                echo "Create the Docker Image"
                sh "docker build -t ${params.CONTAINER_NAME}-redis:1.0 ."
            }
        }
        stage("Clean Up"){
            when{
                expression{
                    params.CLEAR_DOCKER == true
                }
            }
            steps{
               sh "docker rm -f \$(docker ps -f 'publish=${PORT}') || echo 'No Running Containers on PORT: ${PORT} to remove'"
            }
        }
        stage("Deploy"){
            steps{
                sh "docker run --name ${params.CONTAINER_NAME} -p ${PORT}:6379 -d --restart unless-stopped ${params.CONTAINER_NAME}-redis:1.0"
            }
        }
        stage("Test"){
          when{
                expression{
                    params.RUNTESTS == true
                }
            }
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
        failure{echo "Failure!"}
    }
}