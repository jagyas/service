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
          sh "cat /kaniko/.docker/config.json"
          sh "/kaniko/executor --context `pwd` --destination jagyas/service:latest --destination ${REGISTRY_USER}/${PROJECT}:1"
        }
      }
    }
    stage("Create") {
      when { changeRequest target: "master" }
      steps {
        container("knative") {
          sh "kn service create backend --port 8080"
        }
      }
    }
    stage("Deploy") {
      when { branch "master" }
      steps {
        container("kustomize") {
          sh """
            cd kustomize/overlays/production
            kustomize edit set image ${REGISTRY_USER}/${PROJECT}=${REGISTRY_USER}/${PROJECT}:$BRANCH_NAME-${BUILD_NUMBER}
            kustomize build . | kubectl apply --filename -
          """
        }
      }
    }
  }
}
