pipeline {

  options {
    ansiColor('xterm')
  }

  agent {
    kubernetes {
      yamlFile 'builder.yaml'
    }
  }

  stages {
      stage('Maven build') {
        steps {
          container('maven') {
            script {
              sh 'mvn clean package -B -DskipTests'
            }
          }
        }
      }

    stage('Kaniko Build & Push Image') {
      steps {
        container('kaniko') {
          script {
            sh '''
            /kaniko/executor --dockerfile `pwd`/Dockerfile \
                             --context `pwd` \
                             --destination=harbor.netgsm.com.tr:8443/netgsm-sth/netcrm-test:${BUILD_NUMBER} \
                             --cache \
                             --cache-dir=/cache
            '''
          }
        }
      }
    }

    stage('Deploy App to Kubernetes') {
      steps {
        container('kubectl') {
          withCredentials([file(credentialsId: 'k8s-cluster-config', variable: 'KUBECONFIG')]) {
            sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" netcrm.yaml'
            sh 'kubectl apply -f netcrm.yaml'
          }
        }
      }
    }

  }
}
