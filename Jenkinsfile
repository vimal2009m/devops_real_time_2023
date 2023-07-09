pipeline {
    agent any
    environment {
      PATH = "$PATH:/opt/apache-maven-3.9.3/bin"
    }
    
    stages {

        stage('CLEAN WORKSPACE') {
            steps {
                cleanWs()
            }
        }

        stage('CODE CHECKOUT') {
            steps {
                git 'https://github.com/sunnydevops2022/devops_real_time_project_1_english.git'
            }
        }

        stage('MODIFIED IMAGE TAG') {
            steps {
                sh '''
                   export DOCKERHUB_USER=sunnydevops2022
                   cat $WORKSPACE/webapp/src/main/webapp/index.jsp
                   sed -i "s/IMAGE_NAME/$JOB_NAME:v1.$BUILD_ID/g" $WORKSPACE/webapp/src/main/webapp/index.jsp
                   sed -i "s/dockerhub_username/$DOCKERHUB_USER/g" $WORKSPACE/webapp/src/main/webapp/index.jsp
                   cat $WORKSPACE/playbooks/dep_svc.yml
                   echo "#########################################################################"
                   sed -i "s/dockerhub_username/$DOCKERHUB_USER/g" $WORKSPACE/playbooks/dep_svc.yml
                   sed -i "s/image_name:latest/$JOB_NAME:v1.$BUILD_ID/g" $WORKSPACE/playbooks/dep_svc.yml                 
                   cat $WORKSPACE/playbooks/dep_svc.yml                   
                   '''
            }            
        }        
        
        stage('BUILD') {
            steps {
                sh 'mvn clean install package'
            }
        } 
        
        stage('SONAR SCANNER') {
            environment {
            sonar_token = credentials('SONAR_TOKEN')
            sonar_private_ip = credentials('SONAR_PRIVATE_IP')
            }
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectName=$JOB_NAME \
                    -Dsonar.projectKey=$JOB_NAME \
                    -Dsonar.host.url=http://$sonar_private_ip:9000 \
                    -Dsonar.token=$sonar_token'
            }
        }
     
        stage('COPY JAR & DOCKERFILE') {
            steps {
                sh 'ansible-playbook $WORKSPACE/playbooks/create_directory.yml'
            }
        }
        
        stage('PUSH IMAGE ON DOCKERHUB') {
            environment {
            dockerhub_user = credentials('DOCKERHUB_USER')            
            dockerhub_pass = credentials('DOCKERHUB_PASS')
            }    
            steps {
                sh 'ansible-playbook playbooks/push_dockerhub.yml \
                    --extra-vars "JOB_NAME=$JOB_NAME" \
                    --extra-vars "BUILD_ID=$BUILD_ID" \
                    --extra-vars "dockerhub_user=$dockerhub_user" \
                    --extra-vars "dockerhub_pass=$dockerhub_pass"'              
            }
        }
        
        stage('DEPLOYMENT ON EKS') {
            steps {
                sh 'ansible-playbook $WORKSPACE/playbooks/create_pod_on_eks.yml \
                    --extra-vars "JOB_NAME=$JOB_NAME"'
            }            
        }          

    }
}      
