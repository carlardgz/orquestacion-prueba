pipeline {

  environment {
    dockerimagename = "carlarodriguezag/orquestacion-prueba"
    dockerImage = ""
  }

  agent any

  stages {

    stage('Checkout Source') { 
      steps {
        git credentialsId: 'demo_github', url: 'https://github.com/carlardgz/orquestacion-prueba.git', branch:'main'
      }
    }

    stage('Build image') {
      steps{
	dir('proyecto') {
          script {        
	   dockerImage = docker.build dockerimagename
          }
        }
      }
   }

    stage('Pushing Image') {
      environment {
               registryCredential = 'demo_dockerhub'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }

   //stage('Deploying App to Kubernetes') {
   //  steps {
   //    script {
   //      //kubernetesDeploy(configs: "deployment-service-simplesaml.yaml", kubeconfigId: "kuberhaep")
   //       //sh 'microk8s.kubectl rollout restart prueba-gha'
   //     }        
   //   }
   // }

   stage('Restarting POD'){
   steps{
    sshagent(['rodriguezssh'])
    {
     sh 'cd proyecto && scp -r -o StrictHostKeyChecking=no deployment.yaml digesetuser@148.213.1.131:/home/digesetuser/'
      script{
        try{
           sh 'ssh digesetuser@148.213.1.131 microk8s.kubectl apply -f deployment.yaml --kubeconfig=/home/digesetuser/.kube/config'
           sh 'ssh digesetuser@148.213.1.131 microk8s.kubectl rollout restart deployment proyecto --kubeconfig=/home/digesetuser/.kube/config' 
           sh 'ssh digesetuser@148.213.1.131 microk8s.kubectl rollout status deployment proyecto --kubeconfig=/home/digesetuser/.kube/config'
          }catch(error)
       {}
     }
    }
  }
 }
}
}

