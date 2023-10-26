pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
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
    }    
    
    stage('Vulnerability Scan - Docker') {
      steps {
        parallel(
          "Dependency Scan": {
            sh "mvn dependency-check:check"
          },
          "Trivy Scan": {
            sh "bash trivy-docker-image-scan.sh"
          }
        )
      }
    }

    stage('SonarQube - SAST') { // new stage
      steps {
        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=devsecops-numeric-application -Dsonar.host.url=https://30012-port-c09592d5c9f449f8.labs.kodekloud.com -Dsonar.login=sqp_e7514aefdd892274fe2912806797b2c8bdc25e85"
      } // change token and address accordinly 
    }

    stage('Docker image build and push') {
      steps {
        sh 'docker build -t docker-registry:5000/java-app:latest .'
        sh 'docker push docker-registry:5000/java-app:latest'
       }
     }
    stage('Kubernetes Deployment - DEV') {
      steps {
        sh "sed -i 's#REPLACE_ME#docker-registry:5000/java-app:latest#g' k8s_deployment_service.yaml"
        sh "kubectl apply -f k8s_deployment_service.yaml"
      }
    }
   }
 }
