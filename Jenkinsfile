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
          },
          "OPA Conftest": {
            sh "/usr/local/bin/conftest test --policy opa-docker-security.rego Dockerfile"
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
      stage('Vulnerability Scan - Kubernetes') {
      steps {
        parallel(
          "OPA Scan": {
            sh '/usr/local/bin/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
          },
          "Kubesec Scan": {
            sh "bash kubesec-scan.sh"
          }
        )
      }
    }
    
    stage('K8S Deployment - DEV') {
      steps {
        parallel(
          "Deployment": {
              sh "bash k8s-deployment.sh"
          },
          "Rollout Status": {
              sh "bash k8s-deployment-rollout-status.sh"
          }
        )
      }
    }
    
