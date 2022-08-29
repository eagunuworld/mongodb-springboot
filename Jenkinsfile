pipeline{
    agent any

       environment {
            DEPLOY = "${env.BRANCH_NAME == "development" || env.BRANCH_NAME == "master" ? "true" : "false"}"
            NAME = "${env.BRANCH_NAME == "master" ? "example" : "example-staging"}"
            def mavenHome =  tool name: "maven:11.0.16", type: "maven"
            def mavenCMD = "${mavenHome}/usr/share/maven"
            VERSION = "${env.BUILD_ID}"
            REGISTRY = 'eagunuworld/eagunu-mongo-db'
            REGISTRY_CREDENTIAL = 'eagunuworld_credentials'
          }

    stages {
         stage('maven build package') {
              steps {
                  sh "${mavenCMD} clean package"
                   }
                }

          stage('Docker Build') {
                steps {
                   sh "docker build -t ${REGISTRY}:${VERSION} ."
                    }
                 }

        stage('Docker Publish') {
              steps {
                   withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                   sh "docker login -u eagunuworld -p ${docker_password}"
                   }
                 sh 'docker push ${REGISTRY}:${VERSION}'
                }
             }

      stage('configMap And Secret Ref') {
              steps {
                  withCredentials([kubeconfigFile(credentialsId: 'my-configurations', variable: 'KUBECONFIG')]) {
                    sh "kubectl apply -f cm-demo.yml"
                        }
                      }
                 }

      stage('Deployment in Kubernetes clusters') {
              steps {
                  withCredentials([kubeconfigFile(credentialsId: 'my-configurations', variable: 'KUBECONFIG')]) {
                  sh "helm upgrade --install --force --set name=${NAME} --set image.tag=${VERSION} frontend frontend/"
                    }
               }
          }

        stage('Helm Version Deployment Releases') {
                  steps {
                      withCredentials([kubeconfigFile(credentialsId: 'my-configurations', variable: 'KUBECONFIG')]) {
                      sh "helm list"
                    }
              }
          }
    }
}