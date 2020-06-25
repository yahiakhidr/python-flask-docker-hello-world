pipeline {
    agent {
    kubernetes {
      yaml """
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker
  volumes:
    - name: docker-config
      configMap:
        name: docker-config
"""
    }
  }
    environment {
        APP="test-app"
        PHASE=branchToConfig(BRANCH_NAME)
        ECR="647736210332.dkr.ecr.eu-west-1.amazonaws.com"
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
                /kaniko/executor --cache=true --cache-repo=${ECR}/${APP}/${PHASE} --dockerfile `pwd`/Dockerfile --context `pwd` --destination=${ECR}/${APP}/${PHASE}:latest --destination=${ECR}/${APP}/${PHASE}:${GIT_COMMIT} --destination=${ECR}/${APP}/${PHASE}:v$BUILD_NUMBER
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
            agent {
                label 'slave'
            }
            options {
                skipDefaultCheckout true
            }
            when {
                 expression { BRANCH_NAME ==~ /(master)/ }
            }
            steps {
                catchError {
                sh "kubectl set image deployment ${APP} ${APP}=${ECR}/${APP}/${PHASE}:v$BUILD_NUMBER"
                }
            }
        }

        stage('Verifying') {
            agent {
                label 'slave'
            }
            options {
                skipDefaultCheckout true
            }
            when {
                 expression { BRANCH_NAME ==~ /(master)/ }
            }
            steps {
            sh "kubectl rollout status deployment ${APP}"
            }
        }
    }
}

def branchToConfig(branch) {
     script {
        result = "NULL"
        if (branch == 'master') {
        result = "default"
        }
        return result

}
}