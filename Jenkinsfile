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
          sh "/kaniko/executor --context `pwd` --destination ${REGISTRY_USER}/${PROJECT}:latest"
        }
      }
    }
    stage("Test") {
      when { changeRequest target: "master" }
      steps {
        container("kustomize") {
          sh """
            set +e
            kubectl create namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}
            set -e
            cd kustomize/overlays/preview
            kustomize edit set namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}
            kustomize edit set image $REGISTRY_USER/$PROJECT=$REGISTRY_USER/$PROJECT:${env.BRANCH_NAME.toLowerCase()}-$BUILD_NUMBER
            cat ingress.yaml | sed -e "s@host: @host: ${env.BRANCH_NAME.toLowerCase()}@g" | tee ingress.yaml
            kustomize build . | kubectl apply --filename -
            kubectl --namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()} rollout status deployment jenkins-demo
          """
          sh "curl http://${env.BRANCH_NAME.toLowerCase()}$PROJECT.3.67.0.140.nip.io"
          sh "kubectl delete namespace $PROJECT-${env.BRANCH_NAME.toLowerCase()}"
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
