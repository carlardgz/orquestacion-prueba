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
        script {
          dockerImage = docker.build /proyecto/dockerimagename
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
    sshagent(['kubernetessh'])
    {
     sh 'scp -r -o StrictHostKeyChecking=no /proyecto/deployment.yaml digesetuser@148.213.5.79:/home/digesetuser/'
      script{
        try{
           sh 'ssh digesetuser@148.213.5.79 microk8s.kubectl apply -f  /proyecto/deployment.yaml --kubeconfig=/home/digesetuser/.kube/config'
           sh 'ssh digesetuser@148.213.5.79 microk8s.kubectl rollout restart deployment proyecto --kubeconfig=/home/digesetuser/.kube/config' 
           sh 'ssh digesetuser@148.213.5.79 microk8s.kubectl rollout status deployment proyecto --kubeconfig=/home/digesetuser/.kube/config'
          }catch(error)
       {}
     }
    }
  }
 }
}
  post{
            success{
            //slackSend "Build deployed successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            slackSend channel: 'prueba_pipeline_haep', color: 'good', failOnError: true, message: "${custom_msg()}", teamDomain: 'universidadde-bea3869', tokenCredentialId: 'tokenslack'
                  
           }
         }
}

  def custom_msg()
  {
  def JENKINS_URL= "148.213.5.79:8080"
  def JOB_NAME = env.JOB_NAME
  def BUILD_ID= env.BUILD_ID
  def JENKINS_LOG= " DEPLOY LOG: Job [${env.JOB_NAME}] Logs path: ${JENKINS_URL}/job/${JOB_NAME}/${BUILD_ID}/consoleText"
  return JENKINS_LOG
  }
