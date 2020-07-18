pipeline {
   agent any

   stages {
      stage('Build Project') {
         steps {
            '''
            echo 'Build Project'
            cd worker
            mvn install
            '''
         }
      }
   }
}
