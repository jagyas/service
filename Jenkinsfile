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
          sh """/kaniko/executor --context `pwd` --insecure --skip-tls-verify --destination jagyas/service:latest --destination ${REGISTRY_USER}/${PROJECT}:1  --cache=true --cache-copy-layers --cache-repo=jagyas/cache"""
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
