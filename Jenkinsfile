pipeline {
     agent {
         label 'master'
     }
        environment {
        //once you sign up for Docker hub, use that user_id here
        registry = "your_docker_hub_user_id/mypython-app"
        //- update your credentials ID after creating credentials for connecting to Docker Hub
        //registryCredential = 'dockerhub'
        registryCredential = 'git-rino.raimato'
        dockerImage = ''
        AZURE_SUBSCRIPTION_ID = '1a19f6fa-bfc5-48f2-9649-167ecc033cbb'
        AZURE_TENANT_ID = '4db0e941-360c-4031-aa90-f768a23bfc7b'
        CONTAINER_REGISTRY = 'akspipeline'
        RESOURCE_GROUP = 'rg-pipeline-containerRegistry'
        REPO = "nginx"
        IMAGE_NAME = "nginx"
        TAG = "latest"
        TAGProgressive = "${env.BUILD_ID}"
    }
    stages {

        stage ('checkout') {
            steps {
            //checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/akannan1087/myPythonDockerRepo']]])
            //checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-rino.raimato', url: 'https://github.com/guerai/pipeline']]])
            checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/guerai/pipeline']]])
            }
        }
       

        stage('Credentials') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                            sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
                            sh 'az account set -s $AZURE_SUBSCRIPTION_ID'
                            //sh 'az acr login --name $CONTAINER_REGISTRY --resource-group $RESOURCE_GROUP'
                            
                            //sh 'az acr login --name $CONTAINER_REGISTRY'
                            sh 'az acr login -n $CONTAINER_REGISTRY --expose-token'
                            //sh 'az acr build --image $REPO/$IMAGE_NAME:$TAG --registry $CONTAINER_REGISTRY --file Dockerfile . '
                        }
            }
        }
        
        
        stage (' Build docker latest image'){
                steps {
              withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                            sh 'az acr build --image $REPO/$IMAGE_NAME:$TAG --registry $CONTAINER_REGISTRY --file Dockerfile . '
                
              
              }
           }
        }
        
        stage('Build docker image with version') {
            steps {
                
              withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                            sh 'az acr build --image $REPO/$IMAGE_NAME:$TAGProgressive --registry $CONTAINER_REGISTRY --file Dockerfile . '
                
              
              }
           }
        }
        
       stage('Apply Kubernetes files') {
          steps {
             withKubeConfig([credentialsId: 'jenkins-agent-test-environment', serverUrl: 'https://aks-04-dns-28379838.hcp.westeurope.azmk8s.io']) {
              sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
              sh 'chmod u+x ./kubectl'       
              sh './kubectl apply -f nginx.yaml -n test-environment'
             }
          }
       }
         // Uploading Docker images into Docker Hub
    //stage('Upload Im
    // steps{   
    //     script {
    //        docker.withRegistry( '', registryCredential ) {
    //        dockerImage.push()
    //        }
    //    }
    //  }
    //}
 }
}
