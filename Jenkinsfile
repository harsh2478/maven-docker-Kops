pipeline{
    
    agent {
        label "build-system"
    }

    stages{
        stage("SCM") {
            steps{
                git branch: 'main', url: 'https://github.com/harsh2478/maven-docker-Kops.git'
            }
        }
        
        stage("Maven Build"){
            steps{
                  sh 'mvn clean package'
            }
        }
        
        stage("Docker Build"){
            steps {
                sh "sudo docker build -t kanha05/java-web-app:${BUILD_NUMBER} ."
            }
        }
        
        stage ("Pushing image to docker hub"){
            steps{
                withCredentials([string(credentialsId: 'DockerHub', variable: 'passwd')]) {
                    sh "sudo docker login -u kanha05 -p $passwd "
                    sh "sudo docker push kanha05/java-web-app:${BUILD_NUMBER}"
                }
            }
        }
        
        stage("Deploying site in QAT"){
            steps{
                sshagent(['IAM_harsh']) {
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@35.93.32.230 sudo docker run -dit --name TestingApp-${BUILD_NUMBER} -p ${BUILD_NUMBER}:8080  kanha05/java-web-app:${BUILD_NUMBER} "
                }
            }
        }
        
        stage("QAT Testing"){
            steps{
                retry(20){
                    sh "curl --silent http://35.93.32.230:${BUILD_NUMBER}/ |  grep KOPS "
                }
            }
        }
        
        stage("Approving"){
            steps{
                 script {
                    Boolean userInput = input(id: 'Proceed1', message: 'Ready To Go?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
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

	stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "maven-docker-Kops"
                GIT_USER_NAME = "harsh2478"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "harsh.hg2005@gmail.com"
                        git config user.name "Harsh Gupta"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yml
                        git add deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }


	stage("Deploying in Production ") {
            steps{
                sshagent(['IAM_harsh']) {
                    sh "ssh ec2-user@54.218.229.181 sudo wget https://raw.githubusercontent.com/harsh2478/maven-docker-Kops/main/deployment.yml"
                    sh "ssh ec2-user@54.218.229.181 sudo kubectl  apply -f deployment.yml"
		    sh "ssh ec2-user@54.218.229.181 sudo wget https://raw.githubusercontent.com/harsh2478/maven-docker-Kops/main/service.yml"
		    sh "ssh ec2-user@54.218.229.181 sudo kubectl  apply -f service.yml"
		}
	    }

	}
    }
    
    post {
         always {
             echo "Hey from Jenkins"
         }
         success {
              echo "The job ran successfully"
         }
         unstable {
              echo "Gear up ! The build is unstable. Try to fix it"
         }
         failure {
             echo "OMG ! The build failed"
             
         }
     }
}
