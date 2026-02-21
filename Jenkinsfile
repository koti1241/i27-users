// Calling the shared library

/*@Library("com.i27academy.slb@main") _
k8sPipeline(
    appName: 'user'
) */

//code
pipeline {
    agent{
        label 'k8s-slave'
    }
    // tools configured in jenkins master
    //maven download and extract  in jenkins slave under /opt
    //java download and extract in jenkins slave under /opt
    //configured the java and maven in jenkins master under tools 
    parameters{
        choice(name: 'scannerOnly' ,
           choices: 'no\nyes' ,
           description : "this will only scan the code"
        )
        choice(name: 'unitTestOnly' ,
           choices: 'no\nyes' ,
           description : "this will do the unit test"
        )
        choice(name: 'buildOnly' ,
           choices: 'no\nyes' ,
           description : "this will only build the application"
        )
        choice(name: 'dockerPush' ,
           choices: 'no\nyes' ,
           description : "this will  trigger app build, docker build and docker push"
        )
        choice(name: 'deploytoDev' ,
           choices: 'no\nyes' ,
           description : "this will  deploy to dev only"
        )
        choice(name: 'deploytoTest' ,
           choices: 'no\nyes' ,
           description : "this will  deploy to test only"
        )
        choice(name: 'deploytoStage' ,
           choices: 'no\nyes' ,
           description : "this will  deploy to stage only"
        )
        choice(name: 'deploytoProd' ,
           choices: 'no\nyes' ,
           description : "this will  deploy to Prod only"
        )
    }
    tools{
        maven 'Maven-3.8.9'
        jdk 'JDK-17'
    }
    environment{
        APPLICATION_NAME = 'user'
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/koti3355"
        //dockerhub_cred
        DOCKER_CRED = credentials('dockerhub_cred') //dockerhub_cred is id from jenkins -->manage -->credentials
    }
    stages{
        stage('Build'){
            when {
                anyOf {
                    expression {
                        params.dockerPush == 'yes'
                        params.buildOnly == 'yes'
                    }
                }
            }
            steps{
                echo "Build the ${env.APPLICATION_NAME} application"
                sh 'mvn clean package -Dskiptests=true'

                //mvn clean package -Dskiptests=true     //  compile the unit test but does not run the unit test while build the source code (Test classes are still built)
                // mvn clean package -Dmaven.test.skip=true    // skip the complete unit test while build the source code 
                archive 'target/*.jar'
            }
        }
        stage('Unit Test'){
            when {
                anyOf {
                    expression {
                        params.dockerPush == 'yes'
                        params.buildOnly == 'yes'
                    }
                }
            }
            steps{
                echo "############Perform the unit test for ${env.APPLICATION_Name} application##############"
                sh 'mvn test'
            }
        }
        stage('sonar scanning'){
            when{
                anyOf {
                    expression{
                        params.dockerPush == 'yes'
                        params.buildOnly == 'yes'
                        params.scannerOnly == 'yes'
                        
                    }
                }
            }
            steps{
                //quality gates need to implemented in this stage
                //before execute or wirte the code, make sure sonarquce scanner plugin should be installed
                // sonar details are been configured in the Manage jenkins >system
                withSonarQubeEnv('SonarQube') {// SonarQube is the name we have configured in manage jenkins > system >sonarqube, it should match exactly
                
                    sh """
                      echo "scanning the code"
                        mvn clean verify \
                        org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=i27-Eureka

                    """
                }
            }
        }
        stage('Quality Gate') {
            when{
                anyOf {
                    expression{
                        params.dockerPush == 'yes'
                        params.buildOnly == 'yes'
                        params.scannerOnly == 'yes'
                        
                    }
                }
            }
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline failed due to SonarQube Quality Gate: ${qg.status}"
                            }
                       }
                   }
               }
           }
        /*stage('BuildFormat'){
            steps{
                script{
                    //existing:i27-eureka-0.0.1-SNAPSHOT.jar
                    //Destination: i27-eureka-buildnumber-branchname.packagingjar
                    sh """
                    echo "Testing JAR source:i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    echo "Testing Destination Jar:i27-${env.APPLICATION_Name}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"

                    """
                }
            }
        }*/
        stage('DockerBuild'){
            when{
                anyOf{
                    expression{
                        params.dockerPush == 'yes'
                        
                    }
                }
            }
            steps{
                script{

                    dockerBuildAndPush().call()
                   /* sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                    echo"###### building the docker images ############# "
                   sh "docker build --no-cache --build-arg Jar_Source=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t  ${DOCKER_HUB}/${env.APPLICATION_Name}:${GIT_COMMIT} ./.cicd/" //GIT_COMMIT is used to dynamically fetch the latest commit
                   echo "Login to docker registery"
                   sh "docker login -u ${DOCKER_CRED_USR} -p ${DOCKER_CRED_PSW}"
                   echo"############docker push ###################"
                   sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_Name}:${GIT_COMMIT} "
                   */

                }
            }
        }
        stage('Deploy to dev'){
             when{
                anyOf{
                    expression{
                        params.deploytoDev == 'yes'
                        
                    }
                }
            }
            steps{
               
                script{
                    imageValidation().call()

                    dockerDeploy('dev', '8232', '8232').call()
                }
            }
        }
        stage('Deploy to tst'){
            when{
                anyOf{
                    expression{
                        params.deploytoTest == 'yes'
                        
                    }
                }
            }
            steps{
                script{

                   imageValidation().call()
                   dockerDeploy('tst', '3761', '8232').call()
                }
            }
        }
        
        stage('Deploy to stg'){
            when{
                allOf{
                    anyOf{
                        expression{
                            params.deploytoStage == 'yes'
                        }
                   }
                    anyOf{
                        branch 'release/*'
                        tag pattern: "v\\d{1,2}.\\d{1,2}.\\d{1,2}", comparator: "REGEXP"  //v1.2.3
                        }
                }
            }
            steps{
                script{
                    imageValidation().call()

                    dockerDeploy('stg', '4761', '8232').call()
                }
            }
        }
        stage('Deploy to prd'){
            when{
                allOf{
                    anyOf{
                        expression{
                        params.deploytoProd == 'yes'
                        
                        }
                    }
                }
                    anyOf{
                        tag pattern: "v\\d{1,2}.\\d{1,2}.\\d{1,2}", comparator: "REGEXP"  //v1.2.3
                        }
            }
        
            steps{
                timeout(time:300, unit:'SECONDS'){
                    input message: "Deploying to ${env.APPLICATION_NAME} into the production", ok: 'yes', submitter: 'sadwi,admin1'

                }
                script{

                    dockerDeploy('prd', '5671', '8232').call()
                }
                /*echo "*********deploying to dev environment**************"
                withCredentials([usernamePassword(credentialsId: 'docker_vm', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                    // some block
                    // we will communicate to server
                    script{
                        try {
                            sh """
                            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USER_NAME@$dev_ip" '
                                docker container stop "${APPLICATION_NAME}-dev1" || true;
                                docker container rm "${APPLICATION_NAME}-dev1" || true
                            '
                            """

                        }
                        catch(err){
                            echo "Error caught: $err"
                        }

                        // sshpass -p !4u2tryhack ssh -o StrictHostKeyChecking=no username@host.example.com
                       sh """
                            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no $USER_NAME@$dev_ip '
                            docker container run -dit -p 8761:8761 \
                                --name ${env.APPLICATION_NAME}-dev1 \
                                ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} &&
                            docker images
                            '
                            """
                            */

                          ///docker container run -dit -p 8761:8761 --name eureka-dev1 ${env.DOCKER_HUB}/${env.APPLICATION_Name}:${GIT_COMMIT};
                         //docker images
            }
            

         }
    }
}

        
        
        
    
    
    //method for docker build and push
    def dockerBuildAndPush()
    {
        return{
             echo"###### building the docker images ############# "
             sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
             sh "docker build --no-cache --build-arg Jar_Source=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t  ${DOCKER_HUB}/${env.APPLICATION_Name}:${GIT_COMMIT} ./.cicd/" //GIT_COMMIT is used to dynamically fetch the latest commit
             echo "Login to docker registery"
             sh "docker login -u ${DOCKER_CRED_USR} -p ${DOCKER_CRED_PSW}"
             echo"############docker push ###################"
             sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_Name}:${GIT_COMMIT} "

        }
            
                       

    }
    //Build the app:
    def buildApp(){
        return{
            echo "Build the ${env.APPLICATION_NAME} application"
            sh 'mvn clean package -Dskiptests=true'


        }
    }
    //image validation
    def imageValidation(){
        return{
            try{
                echo "******Pull the docker image from repo************"
                sh "docker pull ${env.DOCKER_HUB}/${env.APPLICATION_Name}:${GIT_COMMIT}"
                echo " Image pull successfully"
            }
            catch(Exception e){
                echo " OOPS, docker image with this tag is not available in the repo, so bui;d the image"
                buildApp().call()
                dockerBuildAndPush().call()


            }
            
        }
    }
    //method for dcoker deploy for different environment
    def dockerDeploy(envDeploy, host_port, container_port)
    {
        return{
            echo "*********deploying to $envDeploy environment**************"
                withCredentials([usernamePassword(credentialsId: 'docker_vm', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                    // some block
                    // we will communicate to server
                    script{
                        sh """
                          sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no $USER_NAME@$dev_ip '
                          docker rm -f ${env.APPLICATION_NAME}-${envDeploy} 2>/dev/null || true

                          docker run -dit \
                            -p ${host_port}:${container_port} \
                            --name ${env.APPLICATION_NAME}-${envDeploy} \
                                    ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}
                                '
                                """
                        /*try {
                            sh """
                            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no "$USER_NAME@$dev_ip" '
                                docker container stop "${APPLICATION_NAME}-$env" || true;
                                docker container rm "${APPLICATION_NAME}-$env" || true
                            '
                            """

                        }
                        catch(err){
                            echo "Error caught: $err"
                        }

                        // sshpass -p !4u2tryhack ssh -o StrictHostKeyChecking=no username@host.example.com
                       sh """
                            sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no $USER_NAME@$dev_ip '
                            docker container run -dit -p $host_port:$container_port \
                                --name "${env.APPLICATION_NAME}-$env" \
                                ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} &&
                            docker images
                            '
                            """

                        
                           container port remain same 8232, hostport will be change
                           dev hostport 8232
                           tst hostport 3761
                           stg hostport 4761
                           prd hostport 5671 
                        */

            }
       }

    
    }
    }
