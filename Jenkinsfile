pipeline {
  agent {
    kubernetes {
      defaultContainer "shell"
      yamlFile "jenkins-pod.yaml"
    }
  }
  environment {
    PROJECT = "service"
    REGISTRY_USER = "jagyas"
  }
  stages {
    stage("Build") {
      steps {
        container("kaniko") {
          sh "/kaniko/executor --context `pwd`  --destination ${REGISTRY_USER}/${PROJECT}:${env.BRANCH_NAME.toLowerCase()}-${BUILD_NUMBER}"
        }
      }
    }
    stage("Create") {
      steps {
        container("knative") {
          sh "kn service create backend --port 3000"
        }
      }
    }
    stage("Test") {
      when { branch "master" }
      steps {
        container("knative") {
          sh """
            kn service list
          """
        }
      }
    }
  }
}
