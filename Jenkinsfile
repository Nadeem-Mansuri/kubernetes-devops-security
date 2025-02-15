pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              sh "echo Welcome zaira"
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
          withSonarQubeEnv('SonarQube') {
          sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://devsecops.example.local:9000" // -Dsonar.token=sqp_7b20965f43df7559ab19947f6e15fa8e20a464cd"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true 
                 }
              }
          }
      } 

      stage('Vulnerability Scan - Docker') {
        steps {
          sh "mvn dependency-check:check"
        }
        post {
          always {
            dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
          }
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
  }

      // post {
      //   always {
      //     junit 'target/surefire-reports/*.xml'
      //     jacoco execPattern: 'target/jacoco.exec'
      //     pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
      //     dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'

      //   }
      // }
  }

