pipeline
{


    agent any       //Can run on any client nodes /server agent. Here specifying to run anywhere
    tools {
      maven "MAVEN3" //Specify tools to be used exacty as given in tools section of manage jenkins
      jdk "OracleJDK8"
  }

    environment {            // environment variables
        appRegistry = "prakashbabu9327/vpro-webapp-cicd" //this is just a image artifact value to build docker image
        registryCredential = "dockerhub" // credential from manage credentials in jenkins for login to docker hub

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
                        sh 'mvn clean install -DskipTests'
                    }

                }



        stage('Build App Image') {
       steps {

         script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", ".")
             }

     }

    }

    stage('Upload App Image') {
           steps {

             script {
                    docker.withRegistry ('',registryCredential) {
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')

                    }
                 }

         }

        }

    stage('Remove unused docker image') {
          steps{
            sh "docker rmi $appRegistry:$BUILD_NUMBER"
          }
     }

     stage('Kubernetes Deploy') {
     agent { label 'kops-slave-lbl' }
     steps {
     sh "helm upgrade vpro-app-deploy kubeproject-helm/Kube-Helm-Project/Helm/myvpro-chart --install --force --set imgname=$appRegistry:$BUILD_NUMBER  --namespace prod"
     }
// upgrade and force install if vpro-app-deploy is not existing already
 //chart name along with path from home directory :  kubeproject-helm/Kube-Helm-Project/Helm/myvpro-chart
//vpro-app-deploy is name of release

     }




  }
}

