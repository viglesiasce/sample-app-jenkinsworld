def project = 'vic-next-2018-demo'
def appName = 'sample-app'
def imageTag = "gcr.io/${project}/${appName}:${env.BUILD_NUMBER}"

pipeline {
  agent {
    kubernetes {
      label 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: test-image
    image: gcr.io/${project}/test-image:4.0.0
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    stage('Run go test') {
      steps {
        container('test-image') {
          sh """
            ln -s `pwd` /go/src/sample-app
            cd /go/src/sample-app
            go test
          """
        }
      }
    }
    stage('Build and push image with Cloud Build') {
      steps {
        container('gcloud') {
          sh "PYTHONUNBUFFERED=1 gcloud container builds submit -t ${imageTag} ."
          sh "echo \"imageTag = ${imageTag}\" > properties"
          archiveArtifacts "properties"
        }
      }
    }
  }
}
