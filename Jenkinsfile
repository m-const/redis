pipeline {
    
    parameters{
        string(name: 'CONTAINER_NAME', defaultValue: 'redis', description: "Docker Container Name")
        string(name: 'IMAGE_NAME', defaultValue: 'anura', description: "Docker Container Name")
        //choice(name: 'NAME', choices['3','2','1'], description: "container name")
        booleanParam(name: 'RUNTESTS', defaultValue: false, description: "Run Test Section?")
        booleanParam(name: 'CLEAR_DOCKER', defaultValue: true, description: "Force delete other containers running on this port?")

        //set a PW for redis default user
        //string(name: 'REDIS_DEFAULT_USER_PASS', defaultValue: '', description: "REDIS default user password")

        //Set a non-default user (for application)
        //TODO: change this to use a jenkins credential
        string(name: 'REDIS_USER', defaultValue: 'app', description: "REDIS USER")
        string(name: 'REDIS_PASS', defaultValue: 'pass', description: "REDIS PASS")
        string(name: 'REDIS_PERMISSIONS', defaultValue: 'allkeys allchannels allcommands', description: "Redis ACL permissions to apply to app user.")
    }
    environment{
       PORT="6379"
       //this should be the same temp password in the redis.conf file "requirepass"
       REDIS_INIT_PASS="temppass"
       AGENT_IP="192.168.0.126"

        DOCKER_IMAGE_VERSION="1.2"
    }
    agent { label 'docker' } 
    stages{
        stage("Build"){
            steps{
                echo "Create the Docker Image"
                sh "docker build -t ${params.IMAGE_NAME}-redis:${DOCKER_IMAGE_VERSION} ."
            }
        }
        stage("Clean Up"){
            when{
                expression{
                    params.CLEAR_DOCKER == true
                }
            }
            steps{
                sh "docker rm -f \$(docker ps -q -f 'publish=${PORT}') || echo 'No Running Containers on PORT: ${PORT} to remove'"
            }
        }
        stage("Deploy"){
            steps{
                sh "docker run --name ${params.CONTAINER_NAME} -p ${PORT}:6379 -d --restart unless-stopped ${params.IMAGE_NAME}-redis:${DOCKER_IMAGE_VERSION}"
                sh "docker ps -q -f 'status=running' -f 'publish=${PORT}'"
                
                sh "docker exec -d ${params.CONTAINER_NAME} sed -i 's/BUILDUSER/${params.REDIS_USER}/g' /etc/redis/users.acl"
                sh "docker exec -d ${params.CONTAINER_NAME} sed -i 's/nocommands/''${params.REDIS_PERMISSIONS}''/g' /etc/redis/users.acl"
                sh "docker exec -d ${params.CONTAINER_NAME} sed -i 's/BUILDPWD/${params.REDIS_PASS}/g' /etc/redis/users.acl"

               sh "docker restart ${params.CONTAINER_NAME}"
            }
        }

        //TODO: add new useres to users.acl
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