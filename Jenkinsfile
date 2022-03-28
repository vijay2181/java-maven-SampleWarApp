#!groovy

pipeline {
  agent any
  tools {
  maven 'MAVEN' 
  }
  
  parameters {

        choice(name: 'BUILD', choices: ['Yes', 'No'], description: 'Pick Yes or No')
		string(name: "WAR_FILE_PATH", defaultValue: "/var/lib/jenkins/workspace/final-project/target/*SNAPSHOT.war", trim: false, description: "Provide war file job path in jenkins")
		choice(name: "ENVIRONMENT", choices: ["", "Dev", "Test", "PROD"], description: 'Choose an environment to deploy')		
		string(name: "ANSIBLE_IP", defaultValue: "18.212.36.144", trim: false, description: "Provide Ansible Server IP-ADDRESS")
		password(name: "ANSIBLE_USER_PASSWORD", defaultValue: "vijay", description: "Provide Ansible server user password")
        
    }
    stages {
      
	  stage('environment-setting') {
             steps {
                  script {
                       env.BUILD = "Yes"  // Setting env variable for Build 
					   env.USER = 'Vijay'
                             //Generally it is possible to use Groovy’s conditionals in a declarative syntax, when we use a script step.					   
                             }                     							 
                       }
                 }
				 
	  	  
      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vijay2181/java-maven-SampleWarApp.git']]])
        }
      }
	  
	  
	   stage ('Build')  {
	  
	               
		  when {
                // Only build if "BUILD" value is 'Yes'
                expression { params.BUILD == 'Yes' }
            }
		  steps {
                echo "Hello, Vijay!"
				sh "mvn clean install"
				sh "scp -o StrictHostKeyChecking=no ${params.WAR_FILE_PATH} vijay@${params.ANSIBLE_IP}:/opt/artifact"
				
					
            }
          	   
      }
	        
       stage("Deploy") {
            parallel {                     //parallel is a switch case
                stage("Dev") {
                    when {
                     allOf {
                         expression { params.BUILD == 'Yes' }
                         expression { params.ENVIRONMENT == 'Dev' }        //two conditions must satisfy
                        }
						}
                    steps {
                        echo "Executing Ansible playbook on remote Ansible Host from jenkins Server"
                        sh "sshpass -v -p '${params.ANSIBLE_USER_PASSWORD}' ssh 'vijay@${params.ANSIBLE_IP}' 'ansible-playbook /etc/ansible/playbooks/copy-war.yml --limit webserver1' "
                    }
                }
                stage("Test") {
                    when {
                     allOf {
                         expression { params.BUILD == 'Yes' }
                         expression { params.ENVIRONMENT == 'Test' }
                        }
						}
                    steps {
                        echo "Executing Ansible playbook on remote Ansible Host from jenkins Server"
                        sh "sshpass -v -p '${params.ANSIBLE_USER_PASSWORD}' ssh 'vijay@${params.ANSIBLE_IP}' 'ansible-playbook /etc/ansible/playbooks/copy-war.yml --limit webserver2' "
                    }
                }
                stage("PROD") {
                    when { expression { params.BUILD == 'Yes' }
                          expression { params.ENVIRONMENT == 'Prod'}
                         }
                    steps {
                        sh "echo 'vijay'"
                    }		
			    }
			}
		}
	   
       } 
         
   } 
