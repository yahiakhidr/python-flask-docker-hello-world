pipeline {
    agent {
    kubernetes {
      yaml """
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:1.20.8
    imagePullPolicy: Always
    command:
    - cat
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - cat
    tty: true
"""
    }
  }
    environment {
        APP="test-app"
        PHASE=branchToConfig(BRANCH_NAME)
        REG="yahiakhidr/python-flask-docker-hello-world"
    }

    stages {
        
        stage("Build with Kaniko") {
            when {
                 expression { BRANCH_NAME ==~ /(master)/ }
            }
            steps {
                catchError {
                container(name: 'kaniko') {
                sh '''
                /kaniko/executor --cache=true --cache-repo=${REG} --dockerfile `pwd`/Dockerfile --context `pwd` --destination=${REG}:v$BUILD_NUMBER
                '''
                }
                }
            }
           post {
               success {
                   echo 'Compile Stage Successful'
               }
               failure {
                   echo 'Compile Stage Failed'
                   sh "exit 1"
               }

           }
        }

        stage('Deploy') {

            when {
                 expression { BRANCH_NAME ==~ /(master)/ }
            }
            steps{
                script{
                    container(name: 'kubectl') {
                         withKubeConfig([credentialsId:kube_configs]) {
                            sh "kubectl set image deployment ${APP} ${APP}=${REG}:v$BUILD_NUMBER --namespace ${PHASE}"
                            sh "kubectl rollout status deployment ${APP} --namespace ${PHASE}"
                        }
                    }
                }
            }
        }
    }
}

def branchToConfig(branch) {
    script {
    result = "NULL"
    if (branch == 'master') {
        result = "mendix"
    }
        return result
    }
}