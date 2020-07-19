pipeline {
   agent any

   stages {
      stage('Build Project') {
         steps {
            sh '''
            echo 'Build Project'
            cd worker
            mvn clean install
            '''
         }
      }
      stage('Install docker'){
         steps{
            sh '''
            echo "Install Docker"
            sudo apt-get remove docker docker-engine docker.io containerd runc
            curl -fsSL https://get.docker.com -o get-docker.sh
            sudo sh get-docker.sh
            '''
         }
      }
      stage('Switch User'){
         steps{
            sh '''
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
      }
   }
}
