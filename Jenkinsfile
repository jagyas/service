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
      environment {
        PATH = "/busybox:/kaniko:$PATH"
      }
      steps {
        container(name: 'kaniko', shell: '/busybox/sh') {

          writeFile file: "Dockerfile", text: """
            FROM jenkins/agent
            MAINTAINER CloudBees Support Team <dse-team@cloudbees.com>
            RUN mkdir /home/jenkins/.m2
          """

          sh '''#!/busybox/sh
            /kaniko/executor --context `pwd`  --destination ${REGISTRY_USER}/${PROJECT}:${env.BRANCH_NAME.toLowerCase()}-${BUILD_NUMBER}
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
