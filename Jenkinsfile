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
        container(name: 'kaniko') {
          sh '''
          kubectl delete pod kaniko -n jenkins --ignore-not-found=true
          template=`cat "kaniko-pod.yaml" | sed "s#$jagyas/service:0.0.[0-9]\\+#jagyas/service:0.0.${BUILD_NUMBER}#"`
          echo "$template" | kubectl apply -f -
          sleep 10
          kubectl logs -l pod=kanikp -n jenkins
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
