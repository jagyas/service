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
        container(name: 'kaniko', shell: '/busybox/sh') {
          sh '''
          ls
          kubectl apply -f kaniko-pod.yaml
          '''
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
