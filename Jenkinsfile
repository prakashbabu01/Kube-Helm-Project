pipeline
{


    agent any       //Can run on any client nodes /server agent. Here specifying to run anywhere
    tools {
      maven "MAVEN3" //Specify tools to be used exacty as given in tools section of manage jenkins
      jdk "OracleJDK8"
  }

    environment {            //connect to aws ECR and these are environment variables
        registryCredential = 'ecr:us-east-1:awscredsjenkins_awscreds'  // jenkins aws credential to access from jenkins to aws stored in jenkins
        appRegistry = "107440719127.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg" // its registyurlname/repo name in awr.
        vprofileRegistry = "https://107440719127.dkr.ecr.us-east-1.amazonaws.com"  // registryurl name from above
        cluster = "vprofile_container_service_cluster"
        service = "vprofile-ecs-svc"
    }

    stages
  {
        stage('Fetch code') {  //Names of stages are like comments, can be anything
          steps{
              git branch: 'main', url:'https://github.com/prakashbabu01/Kube-Helm-Project.git'
          }
        }


        stage('Test'){
            steps {
                sh 'mvn test'  //
            }

        }



        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'  //scans code for some be best practives and writes to xml file checkstyle-result.xml
            }
        }

        stage('Build'){
                    steps {
                        sh 'mvn install -DskipTests'  //
                    }

                }



        stage('Build App Image') {
       steps {

         script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
             }

     }

    }

    stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
     }

stage('Deploy to ecs') {
          steps {
        withAWS(credentials: 'awscredsjenkins_awscreds', region: 'us-east-1') {
          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        }
      }
     }

  }
}

