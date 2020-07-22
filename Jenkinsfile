pipeline {
    agent any
    
    tools { 
        maven 'maven' 
        jdk 'jdk1.8' 
    }
    stages {        
      stage('SCM') {
         steps {
            git 'https://github.com/Umeshfarrow/example-voting-app.git'
         }
      }
      stage('Build Project') {
         steps {
            bat '''
            echo 'Build Project'
            cd worker
            mvn clean install
            mvn clean package
            '''
         }
      }
      stage('SonarQube') { 
         steps { 
            bat ''' 
            cd worker 
            mvn sonar:sonar \
           -Dsonar.projectKey=Vote-app-key \
           -Dsonar.host.url=http://localhost:9000 \
           -Dsonar.login=fb245133d74b4674c6c63b892b07c9fe6eba83b9
            '''   	
         }    
      }
    stage('Docker Image'){
        steps{
            bat '''
            echo "Build Images"
            cd vote
            docker build -t umeshfarrow/vote-app .
            cd ../result
            docker build -t umeshfarrow/result-app .
            cd ../worker
            docker build -t umeshfarrow/worker-app .
            echo "Push to repository"
             '''
            withCredentials([string(credentialsId: 'Docker_repository', variable: 'DockerPassword')]) {
                bat 'docker login -u umeshfarrow --password-stdin $(DockerPassword)'
            }
            bat '''
            docker push umeshfarrow/vote-app
            docker push umeshfarrow/result-app
            docker push umeshfarrow/worker-app
            '''
        }
    }
     stage('Approval') {
         // no agent, so executors are not used up when waiting for approvals
         agent none
         steps {
             script {
                 def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'rkivisto,admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                 sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
             }
         }
     }
        stage('AWS Connection and Deployment'){
           steps{
               
               sshagent(['AWS EC2']) {
                   bat '''
                   ssh -o StrictHostKeyChecking=no ubuntu@52.3.249.19
                   docker stop vote worker result db redis
                   docker rm vote worker result db redis
                   docker rmi umeshfarrow/worker-app umeshfarrow/result-app umeshfarrow/vote-app
                   docker-compose up -d
                   '''
               }
          /* bat '''
            ssh -tt ubuntu@54.173.227.243 
            yes
            groot
            sshCommand remote: remote, command: "cd /home/ubuntu;
            sudo su;
            docker stop vote worker result db redis;
            docker rm vote worker result db redis;
            docker rmi umeshfarrow/worker-app umeshfarrow/result-app umeshfarrow/vote-app redis postgres;
            docker-compose up -d;
            '''*/
            }
        }
      
        
   /*     
      stage('Install docker'){
         steps{
            sshCommand remote: remote, command: '''
            echo "Install Docker"
            sudo apt-get remove docker docker-engine docker.io containerd runc
            curl -fsSL https://get.docker.com -o get-docker.sh
            sudo sh get-docker.sh
            '''
         }
      }
      stage('Switch User'){
         steps{
            sshCommand remote: remote, command: '''
            echo "Get in to Sudo"
            sudo su
            '''
         }
      }
     stage('Clone repository'){
         steps{
            sh '''
            git clone https://github.com/Umeshfarrow/example-voting-app.git
            '''
         }
      } 
      stage('Build Docker image'){
         steps{
         sh '''
         echo "Build Images"
         cd example-voting-app
         cd vote
         docker build -t umeshfarrow/vote-app .
         cd ../result
         docker build -t umeshfarrow/result-app .
         cd ../worker
         docker build -t umeshfarrow/worker-app .
         '''
         }
      }
      stage('Build Container'){
         steps{
         sh '''
         echo "Remove containers if any"
         docker rm -f redis db vote result worker
         
         echo "Run redis in name redis"
         docker run -d --name=redis redis
         
         echo "Run Postgres in name db"
         docker run -d --name=db -e POSTGRES_HOST_AUTH_METHOD=trust postgres:9.4
         
         echo "Running project..."
         echo "Deploying Container Vote-app on port 5000"
         docker run -d --name=vote -p 5000:80 --link redis:redis umeshfarrow/vote-app
         
         echo "Deploying Container Result-app on port 5001"
         docker run -d --name=result -p 5001:80 --link redis:redis --link db:db umeshfarrow/result-app
         
         echo "Deploying Container worker-app"
         docker run -d --name=worker --link redis:redis --link db:db prabhavagrawal/worker-app
         '''
         }
      }*/
   } 
   post {
        always {
            //archiveArtifacts artifacts: 'generatedFile.log', onlyIfSuccessful: true
          
            emailext attachLog: true,
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                recipientProviders: [developers(), requestor()],
                subject: "Jenkins Build :- ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
        }
    }
}
