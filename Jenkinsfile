pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        maven 'Maven-3.9.14'
        jdk 'JDK-17'
   }
    parameters {
        choice (name: 'scanOnly',
                choices: 'no\nyes',
        )
        choice (name: 'buildOnly',
            choices: 'no\nyes',
        )
        choice (name: 'dockerPush',
            choices: 'no\nyes',
        )
        choice (name: 'deployToDev',
            choices: 'no\nyes',
        )
        choice (name: 'deployToTest',
            choices: 'no\nyes',
        )
        choice (name: 'deployToStage',
            choices: 'no\nyes',
        )
        choice (name: 'deployToProd',
            choices: 'no\nyes',
        )                                            
    }
    environment {
        APPLICATION_NAME = "users"
        SONAR_HOST= 'http://34.57.207.225:9000'
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/sureshindrala"
        DOCKER_CREDS = credentials('docker_creds')
        DOCKER_SERVER= "35.224.229.170"   
    }
    stages {
        stage('************build-stage************************') {
            when {
                anyOf {
                    expression {
                        params.dockerPush == 'yes'
                        params.buildOnly == 'yes'
                    }
                }
            }            
            steps {

                echo "*****Building-${env.APPLICATION_NAME}******************"           
            sh "mvn clean package -DskipTests=true"
            // sh "mvn clean package -Dmaven.test.skip=true"
                archive 'target/*.jar'
            }

            }
        stage('***********************sonar-stage*******************'){
            when {
                expression {
                    params.dockerPush == 'yes'
                    params.buildOnly == 'yes'                    
                    params.scanOnly == 'yes'
                }
            }
            steps {
            echo "*******${env.APPLICATION_NAME}-sonar scaning*************"
            withCredentials([string(credentialsId: 'sonar_creds', variable: 'sonar_creds')]){
                sh """
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=chathura-eureka \
                    -Dsonar.host.url=$SONAR_HOST \
                    -Dsonar.login=$sonar_creds        

                """
            }

            }

        }
        stage('Build Format') {
            when {
                expression {
                    params.dockerPush == 'yes'
                    // params.buildOnly == 'yes'                    
                    // params.scanOnly == 'yes'
                }
            }
            steps {
                    echo "***************************Printing Build Format*****************************"
                    script {
                        dockerBuildandPush().call()
                    }
                    // script {
                    //     sh """
                    //     echo "Testing JAR SOURCE: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    //     echo "Testing JAR Destination Format: i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                    
                    //     """
                    //     sh "cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                    //     sh "ls -la ./.cicd"
                    //     sh "docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd "
                        
                    //     echo "****************** Login to Docker Registry ******************"

                    //     sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                    //     echo "****************** Push Image to Docker Registry ******************"
                    //     sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"                
                    
                    // }
                }
            }
            // stage ('************docker-build and push************************') {
            //     steps{
            //         script{
            //             dockerBuildandPush().call()
            //         }
            //     }
            // }
        stage('Deploy to Dev') {
            when {
                expression {
                    params.deployToDev == 'yes'
                }
            }
            steps {
                // withCredentials([usernamePassword(
                //     credentialsId: 'greesh_creds',
                //     usernameVariable: 'USERNAME',
                //     passwordVariable: 'PASSWORD'
                // )]) {
                    script {
                        imageValidation().call()
                        dockerdeploy('dev', '5761').call()
                        
                        // try {
                        //     // Stop existing container
                        //     sh """
                        //     sshpass -p '${PASSWORD}' ssh -o StrictHostKeyChecking=no ${USERNAME}@${env.DOCKER_SERVER} "docker stop ${env.APPLICATION_NAME} || true"
                        //     sshpass -p '${PASSWORD}' ssh -o StrictHostKeyChecking=no ${USERNAME}@${env.DOCKER_SERVER} "docker rm ${env.APPLICATION_NAME} || true"
                        //     """

                        //     // Run new container
                        //     sh """
                        //     sshpass -p '${PASSWORD}' ssh -o StrictHostKeyChecking=no ${USERNAME}@${env.DOCKER_SERVER} "docker container run -dit -p 8761:8761 --name ${env.APPLICATION_NAME} ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                        //     sshpass -p '${PASSWORD}' ssh -o StrictHostKeyChecking=no ${USERNAME}@${env.DOCKER_SERVER} "docker ps"
                        //     """
                        // } catch (err) {
                        //     echo "Error caught: ${err}"
                        // }
                        // // sh """
                        // //     sshpass -p '${PASSWORD}' ssh -o StrictHostKeyChecking=no ${USERNAME}@${env.DOCKER_SERVER} "docker container run -dit -p 8761:8761 --name ${env.APPLICATION_NAME} ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                        // // """
                    }
                }
            }
        stage('Deploy to Test'){
            when {
                expression {
                    params.deployToTest == 'yes'
                }
            }
            steps {
                script{
                    imageValidation().call()
                    dockerdeploy('tst', '6761').call()
                    }
                }
            }
        stage('Deploy to stage'){
            when {
                allOf {
                anyOf {
                    expression {
                        params.deployToStage == 'yes'
                    }
                }
                anyOf {
                    branch 'release/*'
                    tag pattern: "v\\d{1,2}\\.\\d{1,2}\\.\\d{1,2}", comparator: "REGEXP" // v1.2.3 is the correct one, v123 is the wrong one
                }
            }
            }
            steps {
                script {   
                    imageValidation().call()
                    dockerdeploy('stage', '7761').call()
                }
            }
        }
        stage('Deploy to prod'){
            // when {
            //     expression {
            //         params.deployToProd == 'yes'
            //     }
            // }
            when {
                allOf {
                    anyOf {
                        expression {
                            params.deployToProd == 'yes'
                        }
                    }
                    anyOf {
                        tag pattern: "v\\d{1,2}\\.\\d{1,2}\\.\\d{1,2}", comparator: "REGEXP" // v1.2.3 is the correct one, v123 is the wrong one
                    }
                }
            }
            steps {
                timeout(time: 300, unit: 'SECONDS'){ // SECONDS, MINUTES, HOURs
                     input message: "Deploying to ${env.APPLICATION_NAME} to production ??", ok:'yes', submitter: 'Suresh Indrala'
                }
                script {
                    imageValidation().call()
                    dockerdeploy('prod', '8761').call()
                }                
            }
        }                
    }                        
        
}


def buildApp(){
    return {
        echo "Building ${env.APPLICATION_NAME} Application"
        sh 'mvn clean package -DSkipTests=true'
    }
}



def imageValidation() {
    return {
        println("**************Attempting pull the docker image**********")
        try {
            sh "docker pull ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
            println("*************docker image pulled succesfully*************")
        }
        catch(Exception e) {
            println("*************OOPS..!*****The docker image with this tag is not avaliable in this repo, So creating the Image****")
            buildApp().call()
            dockerBuildandPush().call()

        }
    }
}

def dockerBuildandPush() {
    return {
        script {
            sh """
            echo "Testing JAR SOURCE: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
            echo "Testing JAR Destination Format: i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
        
            """
            sh "cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
            sh "ls -la ./.cicd"
            sh "docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd "
            
            echo "****************** Login to Docker Registry ******************"

            sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
            echo "****************** Push Image to Docker Registry ******************"
            sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"                
        
        }   

               
    }
}

def dockerdeploy(envDeploy,envPort) {
    return{
    withCredentials([usernamePassword(credentialsId: 'greesh_creds', 
        passwordVariable: 'PASSWORD', 
        usernameVariable: 'USERNAME')]) {
        try {
            // Stop existing container
            sh """
            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"${env.DOCKER_SERVER}" "docker stop ${env.APPLICATION_NAME}-${envDeploy} || true"
            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"${env.DOCKER_SERVER}" "docker rm ${env.APPLICATION_NAME}-${envDeploy} || true"
            """

            // Run new container
            sh """
            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"${env.DOCKER_SERVER}" "docker container run -dit -p ${envPort}:8761 --name ${env.APPLICATION_NAME}-${envDeploy} ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"${env.DOCKER_SERVER}" "docker ps"
            """
        } catch (err) {
            echo "Error caught: ${err}"
            }
        }
    }
}

// Container port will be 8761 only, only host port changes
// dev: HostPort = 5761
// tst: HostPort = 6761
// stg: HostPort = 7761
// prod: HostPort = 8761