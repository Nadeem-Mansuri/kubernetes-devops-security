pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }

      stage('Unit Tests') {
        steps {
          sh "mvn test"
        }
        post {
          always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
          }
        }
      }

      stage('Docker Build and Push') {
        steps {
          docker.withRegistry([credentialsId: "docker-hub", url: ""]){
            sh 'printenv'
            sh 'docker build -t ndmkhan068/devsecops/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push ndmkhan068/devsecops/numeric-app:"" $GIT_COMMIT""'
          }
        }
      } 
    }
}