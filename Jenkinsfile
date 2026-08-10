pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        jdk 'java_21'
        maven 'maven_3.6'
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
        APPLICATION_NAME = "user"
        SONAR_HOST = "http://35.196.58.210:9000"
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/sureshindrala"
        DOCKER_CREDS = credentials('docker_creds')
        

    }
    stages {
        stage('build') {
            when {
                anyOf {
                    expression {
                        params.dockerPush == 'yes'
                        params.buildOnly == 'yes'                        
                    }

                }
            }
            steps{
                echo "**********priniting ${env.APPLICATION_NAME}****************"
                sh "mvn clean package -DskipTests=true "
                archive 'target/*.jar'

            }
        }
        stage ('sonar') {
            when {
                anyOf {
                    expression {
                        params.dockerPush == 'yes'
                        params.scanOnly == 'yes'                        
                    }
                }
            }
           steps {          
             echo "******************${APPLICATION_NAME}-*****Sonar stage*******************"
                withCredentials([string(credentialsId: 'sonar_creds', variable: 'sonar_creds')]) {
                    sh """
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar\
                        -Dsonar.projectKey=maven_project \
                        -Dsonar.host.url=$SONAR_HOST\
                        -Dsonar.login=$sonar_creds
                    """ 
                }
           
            }
            
        }
        // stage ('BUILD_FORMAT') {
        //     steps {

        //         script {
        //             echo "Testing JAR SOURCE: chathura-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
        //             echo "Testing Destination Format: chathura-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"


        //         }

        //     }
        // }
        // stage ('Docker_BUILD') {
        //     steps {
        //         script {
        //             echo "************Build Docker Image*****************"
        //             sh "cp ${workspace}/target/chathura-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
        //             sh "ls -la ./.cicd"
        //             sh "docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=chathura-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd "
            
        //         }
        //     }
        // }
        // stage ('Docker Push') {
        //     steps {
        //     script {
        //     echo "*****************building Docker image***********************"
                            
        //         }
        //     }
        // }
        stage ('docker build and push') {
            when {
                anyOf {
                    expression {
                        params.dockerBuildandPush == 'yes'
                        
                    }
                }
            }
            steps{
                script{
                    dockerBuildandPush().call()
                }
            }
        }
        
        // stage ('docker build Dev') {
        //     steps {
        //         echo "*****************Building Docker container-Dev*****************************"
        //         withCredentials([usernamePassword(credentialsId: 'dockervm_greesh_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
        //             script {
        //                 try {
        //                     sh """
        //                         sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"$docker_server_ip" "docker stop ${env.APPLICATION_NAME}-dev "
        //                         sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"$docker_server_ip" "docker rm ${env.APPLICATION_NAME}-dev"
        //                     """
        //                 }
        //                 catch($err){
        //                     echo "Error caught: $err"
        //                 }
        //             sh """    
        //              sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"$docker_server_ip" "docker run -dit -p 8761:8761 --name ${env.APPLICATION_NAME}-dev ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
        //              sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"$docker_server_ip" "docker ps"
        //              """
        //             }
        //         }

        //     }
        // }
        stage('docker deploy-dev') {
            when {
                anyOf {
                    expression {
                        params.deployToDev == 'yes'
                    }
                }
            }
            steps {
                script {
                    imageValidation().call()
                    dockerdeploy('dev','5761').call()
                }
            }
        }
        stage('docker deploy-test') {
            when {
                anyOf{
                    expression{
                        params.deployToTest == 'yes'
                    }
                }
            }
            steps {
                script {
                    imageValidation().call()
                    dockerdeploy('test','6232').call()
                }
            }
        }
        stage('docker deploy-stage') {
            when {
                allOf{
                   anyOf{
                     expression{
                            params.deployToStage == 'yes'
                        }
                    }
                    anyOf {
                        branch 'release/*'
                        tag pattern: "v\\d{1,2}\\.\\d{1,2}\\.\\d{1,2}\\", comparator: "REGEXP"
                    }  
                }

            }
            steps {
                script {
                    imageValidation().call()
                    dockerdeploy('stage','7232').call()
                }
            }
        } 
        stage('docker deploy-prod') {
            when {
                allOf{
                   anyOf{
                        expression{
                            params.deployToProd == 'yes'
                    }
                }
                    anyOf {
                        tag pattern: "v\\d{1,2}\\.\\d{1,2}\\.\\d{1,2}", comparator: "REGEXP" // v1.2.3 is the correct one, v123 is the wrong one
                    }                
                }

            }
            steps {
                script {
                    imageValidation().call()
                    dockerdeploy('prod','8232').call()
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
        echo "*****************building Docker image***********************"
        sh """
            cp ${workspace}/target/chathura-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
            ls -la ./.cicd
            docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=chathura-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}  ./.cicd
            echo "***********Docker login***********************"
            docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW} 
            docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}


        """        
    }
}


def dockerdeploy(envDeploy,envPort) {
    return{
    withCredentials([usernamePassword(credentialsId: 'dockervm_greesh_creds', 
        passwordVariable: 'PASSWORD', 
        usernameVariable: 'USERNAME')]) {
        try {
            // Stop existing container
            sh """
            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"$docker_server_ip" "docker stop ${env.APPLICATION_NAME}-${envDeploy} || true"
            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"$docker_server_ip" "docker rm ${env.APPLICATION_NAME}-${envDeploy} || true"
            """

            // Run new container
            sh """
            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"$docker_server_ip" "docker container run -dit -p ${envPort}:8761 --name ${env.APPLICATION_NAME}-${envDeploy} ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USERNAME"@"$docker_server_ip" "docker ps"
            """
        } catch (err) {
            echo "Error caught: ${err}"
            }
        }
    }
}

//  @Library("com.chathura.slb@main") _ 
//  jfrogPipeline (
//      appName: 'eureka'

//  )


//  @Library("com.chathura.slb@main") _ 
//  k8sPipeline (
//      appName: 'eureka'

//  )




// @Library("com.chathura.slb@main") _ 
// dockerPipeline (
//     appName: 'eureka',
//     devHostPort: '5761',
//     contPort: '8761'
// )
// // L shoud be capital