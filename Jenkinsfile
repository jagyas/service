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
          git clone https://github.com/jagyas/service.git
          sed -i "s#jagyas/service:0.0.[a-zA-Z0-9]\\+#jagyas/service:0.0.${BUILD_NUMBER}#" kaniko-pod.yaml
          kubectl apply -f kaniko-pod.yaml
          sleep 10
          kubectl logs -l pod=kaniko -f -n jenkins
          '''
        }
      }
    }
    stage("Create") {
      steps {
        container("knative") {
          sh "kn service update backend --image jagyas/service:0.0.${BUILD_NUMBER} -n default"
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
