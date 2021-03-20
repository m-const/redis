pipeline {
    
    parameters{
        string(name: 'CONTAINER_NAME', defaultValue: 'anura', description: "Docker Container Name")
        //choice(name: 'NAME', choices['3','2','1'], description: "container name")
        booleanParam(name: 'RUNTESTS', defaultValue: false, description: "Run Test Section?")
        booleanParam(name: 'CLEAR_DOCKER', defaultValue: false, description: "Force delete other containers running on this port?")

        //set a PW for redis default user
        string(name: 'REDIS_DEFAULT_USER_PASS', defaultValue: '', description: "REDIS default user password")

        //Set a non-default user (for application)
        string(name: 'REDIS_USER', defaultValue: '', description: "REDIS USER")
        string(name: 'REDIS_PASS', defaultValue: '', description: "REDIS PASS")
        string(name: 'REDIS_PERMISSIONS', defaultValue: '~* &* +@all', description: "Redis ACL permissions to apply to app user.")
    }
    environment{
       PORT="3001"
       //this should be the same temp password in the redis.conf file "requirepass"
       REDIS_INIT_PASS="temppass"
       AGENT_IP="192.168.0.126"
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
                sh "docker rm -f \$(docker ps -q -f 'publish=${PORT}') || echo 'No Running Containers on PORT: ${PORT} to remove'"
            }
        }
        stage("Deploy"){
            steps{
                sh "docker run --name ${params.CONTAINER_NAME} -p ${PORT}:6379 -d --restart unless-stopped ${params.CONTAINER_NAME}-redis:1.0"
                sh "docker ps -q -f 'status=running' -f 'publish=${PORT}'"
                
                sh '''#!/bin/bash
                 "redis-cli -h "+${AGENT_IP}+" -p "+${PORT}+" AUTH "+${REDIS_INIT_PASS}+" ACL SETUSER default nopass >"+${params.REDIS_DEFAULT_USER_PASS}+""
                 "redis-cli -h "+${AGENT_IP}+" -p "+${PORT}+" AUTH "+${params.REDIS_DEFAULT_USER_PASS}+" ACL SETUSER "+${params.REDIS_USER}+" on >"+${params.REDIS_PASS}+" "+${params.REDIS_PERMISSIONS}+""
                    '''

                //sh "redis-cli -h ${AGENT_IP} -p ${PORT} AUTH ${REDIS_INIT_PASS} ACL SETUSER default nopass >${params.REDIS_DEFAULT_USER_PASS}"
                //sh "redis-cli -h ${AGENT_IP} -p ${PORT} AUTH ${params.REDIS_DEFAULT_USER_PASS} ACL SETUSER ${params.REDIS_USER} on >${params.REDIS_PASS} ${params.REDIS_PERMISSIONS}"
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