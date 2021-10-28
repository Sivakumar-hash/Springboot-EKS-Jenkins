# Springboot-EKS-Jenkins

#Deploy Springboot Microservices App into Amazon EKS Cluster using Jenkins Pipeline | Containerize Springboot App and Deploy into EKS Cluster using Jenkins Pipeline
- Automating builds using Jenkins
- Automating Docker image creation
- Automating Docker image upload into Docker Hub
- Automating Deployments to Kubernetes Cluster

Pre-requistes:


1. Amazon EKS Cluster is setup and running. Click here to learn how to create Amazon EKS cluster.
2. Jenkins Master is up and running
3. Setup Jenkins slave, install docker in it.
4. Docker, Docker pipeline and Kubernetes Continuous Deploy plug-ins are installed in Jenkins
5. Docker hub account setup in https://cloud.docker.com
6. Install kubectl on your instance

Go to master jenkins server
Step1:
Create Credentials for Docker Hub
Go to Jenkins UI, click on Credentials -->
Click on Global credentials
Click on Add Credentials
Now Create an entry for your Docker Hub account. Make sure you enter the ID as dockerhub

Step2:
Create Credentials for Kubernetes Cluster
Click on Add Credentials, use Kubernetes configuration from drop down
execute the below command to get kubeconfig info, copy the entire content of the file:
sudo cat ~/.kube/config
Enter ID as K8S and choose enter directly and paste the above file content and save.

Step3:
Create Maven3 variable under Global tool configuration in Jenkins
Make sure you create Maven3 variable under Global tool configuration

Step4:
set a clusterrole as cluster-admin

By default, clusterrolebinding has system:anonymous set which blocks the cluster access. Execute the following command to set a clusterrole as cluster-admin which will give you the required access.

kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous

Step5:
Create a pipeline in Jenkins
Create a new pipeline job.

Step6:
Copy the pipeline code from below:
Your docker user id should be updated.
your registry credentials ID from Jenkins from step # 1 should be copied
node ("slave") {
  def image
  def mvnHome = tool 'Maven3'
     stage ('checkout') {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/Sivakumar-hash/Springboot-application-code.git']]])      
        }
   
    stage ('Build') {
            sh 'mvn -f MyAwesomeApp/pom.xml clean install'           
        }
        
        stage ('Build') {
            sh 'mvn -f MyAwesomeApp/pom.xml clean install'           
        }
       
       
    stage ('Docker Build') {
         // Build and push image with Jenkins' docker-plugin
            withDockerRegistry([credentialsId: "dockerhub", url: "https://index.docker.io/v1/"]) {
            image = docker.build("akdevopscoaching/mywebapp", "MyAwesomeApp")
            image.push()    
            }
        }

      stage ('K8S Deploy') {
       
                kubernetesDeploy(
                    configs: 'MyAwesomeApp/springboot-lb.yaml',
                    kubeconfigId: 'K8S',
                    enableConfigSubstitution: true
                    )               
        }
    
}


Step7:
Build the pipeline
Once you create the pipeline and changes values per your Docker user id and credentials ID, click on console output to validate

Step8:
Verify deployments to K8S
kubectl get pods
kubectl get deployments
kubectl get services

Step9:
Access SpringBoot App in K8S cluster
Once build is successful, go to browser and enter master or worker node public ip address along with port number mentioned above
http://master_or_worker_node_public_ipaddress:port_no_from_above





