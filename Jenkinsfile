pipeline {
   agent any

   stages {
      stage('Build Project') {
         steps {
            bat '''
            echo 'Build Project'
            cd worker
            mvn install
            '''
         }
      }
   }
}
