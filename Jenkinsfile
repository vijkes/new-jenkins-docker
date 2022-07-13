pipeline {
    
    agent {
        label "linuxbuildnode"
    }
    
    
    stages {
        stage('SCM') {
            steps {
                git 'https://github.com/vijkes/new-jenkins-docker.git'
                
            }
            
        }
        
        stage('Build by Maven Package') {
            steps {
                sh 'mvn clean package'
            }
            
        }
        
        
        stage('Build Docker OWN image') {
            steps {
                sh "sudo docker build -t vijkes/javaweb:${BUILD_TAG}  ."
                //sh 'whoami'
            }
            
        }
        
        
        stage('Push Image to Docker HUB') {
            steps {
                
                withCredentials([string(credentialsId: 'DOCKER_JENKINS_PWD', variable: 'NEWDOCJENTECHCOM')]) {
    // some block
                 sh "sudo docker login -u vijkes -p ${NEWDOCJENTECHCOM}"
                 
                sh "sudo docker push vijkes/javaweb:${BUILD_TAG}"
            }
            
        }
        }
        
        stage('Deploy webAPP in DEV Env') {
            steps {
                sh 'sudo docker rm -f mynewjavaapp'
                sh "sudo docker run -d -p 8080:8080 --name mynewjavaapp  vijkes/javaweb:${BUILD_TAG}"
                //sh 'whoami'
            }
            
        }
        
        
        stage('Deploy webAPP in QA/Test Env') {
            steps {
               
               sshagent(['QA_ENV_SSH_CRED']) {
    
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@3.22.171.22 sudo docker rm -f mynewjavaapp"
                    sh "ssh ec2-user@3.22.171.22 sudo docker run -d -p 8080:8080 --name mynewjavaapp  vijkes/javaweb:${BUILD_TAG}"
                }

            }
            
        }
        
        /*
         stage('QAT Test') {
            steps {
                
                sh 'curl --silent http://3.22.171.22:8080/java-web-app/ |  grep India'
                
                //retry(10) {
                  //  sh 'curl --silent  http://18.224.215.103:8080/java-web-app/ | grep India'
                //}
            
               
            }
        }
          */
        
         
         
        stage('approved') {
            steps {
                
            
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
        
        
         
        
        stage('Deploy webAPP in Prod Env') {
            steps {
               
               sshagent(['QA_ENV_SSH_CRED']) {
    
                    
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@18.221.178.91 sudo docker rm -f mynewjavaapp"
                    sh "ssh ec2-user@18.221.178.91 kubectl version --short --client"
                    sh "ssh  ec2-user@18.221.178.91 sudo kubectl  create    deployment mynewjavaapp  --image=vijkes/javaweb:${BUILD_TAG}"
                    sh "ssh ec2-user@18.221.178.91 sudo wget https://raw.githubusercontent.com/vimallinuxworld13/jenkins-docker-maven-java-webapp/master/webappsvc.yml"
                    sh "ssh ec2-user@18.221.178.91 sudo kubectl  apply -f webappsvc.yml"
                    sh "ssh ec2-user@18.221.178.91 sudo kubectl  scale deployment mynewjavaapp --replicas=5"
                }

            }
            
        } 
        
    
        
    }
    
  
        
     post {
         always {
             echo "You can always see me"
         }
         success {
              echo "I am running because the job ran successfully"
         }
         unstable {
              echo "Gear up ! The build is unstable. Try fix it"
         }
         failure {
             echo "OMG ! The build failed"
             mail bcc: '', body: 'hi check this ..', cc: '', from: '', replyTo: '', subject: 'job ete fail', to: 'vdaga@lwindia.com'
         }
     }

    
    
}
