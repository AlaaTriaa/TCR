pipeline{
agent any
environment {
DOCKERHUB_CREDENTIALS= credentials('DOCKER_HUB_SECRET')  //DOCKER_HUB_SECRET is the ID of the secret you've provided
}

stages{
stage("SCM : Clone code from github"){
steps{
git credentialsId: 'GIT_HUB_CREDENTIALS', branch: 'backend',url: 'https://github.com/rawand123-bot/Talan-Container-Registry'


} //GIT_HUB_CREDENTIALS is the ID of the your github credentials
}
stage(" Continuous build: Build code Using Maven"){
steps{
sh "checkov -f pom.xml  --hard-fail-on HIGH "
sh "mvn -f pom.xml clean package -DskipTests"
}

}
stage ('build docker image from Dockerfile') {
steps{
  
sh "checkov -f Dockerfile  --hard-fail-on HIGH "
sh "docker build -t registry:v1 ."
sh "docker tag registry:v1 alaatriaa/registry:v1 //BUILD NUMBER is a variable provided by jenkins 
//alaatriaa  is the name of my dockerhub repo and registry the name of the built image
echo 'Build Image Completed'
}}


stage('Push Image to DockerHub') {
steps{
sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
echo 'Login Completed' //USR & PSW are provided by jenkins you just have to give your credentials id 
 sh "docker push alaatriaa/registry:v1"
}
}


stage('Deploy the whole application on the kubeadm cluster'){
     
        steps{
            script{
            
        def remote = [:]
        remote.name = 'master' //name of the node
        remote.host = '10.0.0.10' //ip @ of the master node
        remote.user = 'vagrant'
        remote.password = 'vagrant'
        remote.allowAnyHosts = true
       
       //Deploy backend application (springboot)
       
        sh "checkov -f spring-deployment.yml  --hard-fail-on HIGH "
        sshPut remote: remote, from: 'backend.yaml', into: '.' //this will copy the content of deployment files from git repo to your cluster 
        sshCommand remote: remote, command: "kubectl apply -f backend.yaml
        
        //Deploy database
        
        sh "checkov -f database.yaml  --hard-fail-on HIGH "
        sshPut remote: remote, from: 'database.yaml', into: '.'
        sshCommand remote: remote, command: "kubectl apply -f database.yaml"
         
         }
   

}
    }
    stage("wait till pod is created") { //sleep 
           
 steps {
                echo "sleep"
                sleep(time: 60, unit: "SECONDS")
            }
        }
        stage('check if pods are working'){
     
        steps{
            script{
               def remote = [:]
        remote.name = 'master'
        remote.host = '10.0.0.10'
        remote.user = 'vagrant'
        remote.password = 'vagrant'
        remote.allowAnyHosts = true
         try{
         sshCommand remote: remote, command: "kubectl get pods"
           
             } catch(Throwable e){
              echo "Caught ${e.toString()}"
              echo "ERROR GETTING PODS!  "

}
     try{
         sshCommand remote: remote, command: "kubectl get svc"
           
             } catch(Throwable e){
              echo "Caught ${e.toString()}"
              echo "ERROR GETTING SERVICES!"

}

        
         
         
         }
   

}
    }
    }//stages
    
post{ //post stage= after building our stages 
success { //in case our pipeline has succeeded to build all of its stages we will delete the image (memory gain)
print 'Job completed successfully.'
sh "docker rmi -f alaatriaa/registry:v1"
}
failure {
print 'Job failed.' //in failure ,we will send an email to notify the developper 
// notify email on failure

mail to: 'alaeddine.triaa@supcom.tn',
subject: "Failure: ${env.BUILD_TAG}",
body: "Job failed for ${env.JOB_NAME} at ${env.JOB_URL}."

}
always { //after all we have to clean the the docker system
// remove built docker image and prune system
print 'Cleaning up the Docker system.'
sh 'docker system prune -f'
}

}
}
    
//pipeline
