pipeline {
    
    agent {
        label   "Jenkins-slave-linux-1"
    }
    
    stages  {
        stage('SCM')  {
            steps  {
                git 'https://github.com/vijkes/new-jenkins-docker.git'
            }
            
        }
         
        stage('Build on Mvn package')  {
            steps {
                sh 'mvn clean package'
            }
        }    

        stage('Build Docker OWN images') {
            steps {
                
                sh "sudo docker build -t vijkes/javaweb:${BUILD_TAG} ."
                
            }
        }
        
        stage('Push image to Docker') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_JENKINS_PWD', variable: 'NEWDOCJENTECHCOM')]) {
    // some block

            
                sh "sudo docker login -u vijkes -p ${NEWDOCJENTECHCOM}"
                 
                sh "sudo docker push vijkes/javaweb:${BUILD_TAG}"
                
                //sh "sudo docker rmi vijkes/javaweb:${BUILD_TAG}"
               
            }
         }
        }         
    
        stage('Deploy webApp in DEV Env') {
            steps {
                sshagent(['QA_ENV_SSH_CRED']) {

                sh 'ssh -o StrictHostKeyChecking=no ec2-user@13.59.66.87 sudo docker rm -f mynewjavaapp'
                sh "ssh ec2-user@13.59.66.87 sudo docker run -d -p 8080:8080 --name mynewjavaapp  vijkes/javaweb:${BUILD_TAG}"
                
            }
        }
    }    
        
        stage('QAT Test') {
            steps {
                
                retry(10) {
                sh 'curl --silent  http://13.59.66.87:8080/java-web-app/ | grep India'
                }
            } 
            
        }
        
        stage('approved')  {
            steps   {
                
                script {
                    Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                    echo 'userInput: ' + userInput

                    if(userInput == true) {
                    // do action
                    } else {
                        // not do action
                        echo "Action was aborted."
                    }
            
                
            }
        
            }
        }  
        
        
        
        stage('Deploy webApp in Prod Env') {
            steps {
                sshagent(['QA_ENV_SSH_CRED']) {
                
                sh 'ssh -o StrictHostKeyChecking=no ec2-user@3.16.149.213 kubectl delete deployment myjavawebapp'
                sh 'ssh ec2-user@3.16.149.213  kubectl create deployment myjavawebapp --image=vijkes/javaweb:${BUILD_TAG}'
                sh "ssh ec2-user@3.16.149.213 wget https://raw.githubusercontent.com/vimallinuxworld13/jenkins-docker-maven-java-webapp/master/webappsvc.yml"
                sh "ssh ec2-user@3.16.149.213  kubectl apply -f webappsvc.yml"
                sh "ssh ec2-user@3.16.149.213 kubectl scale deployment myjavawebapp --replicas=5"
            }
        }
    }   
    
    }
}
