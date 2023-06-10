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

      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            }
          }
      }
    
      stage('SonarQube - SAST') {
        steps {
          sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://devsecops.example.local:9000 -Dsonar.token=sqp_c1ac73c778f852442adccafad5938f8e4a6f6688'
        }
      }

      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'printenv'
            sh 'docker build -t ndmkhan068/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push ndmkhan068/numeric-app:""$GIT_COMMIT""'
          }
        }
      }
      stage('Kubernetes Deployment - DEV') {
        steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "sed -i 's#replace#ndmkhan068/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
              } 
          }
      }

    //  stage('Deploying numeric-app incremental apps in Kubernetes - DEV') {
    //    steps {
    //          withKubeConfig([credentialsId: 'kubeconfig']) {
    //            sh 'kubectl create deploy node-app --image siddharth67/node-service:v1'
    //            sh 'kubectl -n default expose deploy node-app --name node-service --port 5000'
    //          }
    //    }
    //  }    
  }
}
