pipeline {
    agent { label 'kube' }

    environment {
        // GIT_COMMIT_SHORT = sh(
        //     script: "printf \$(git rev-parse --short ${GIT_COMMIT})",
        //     returnStdout: true
        // )
        // GIT_COMMIT_DATE = sh(
        //     script: "printf \$(git show -s --format=%ci ${GIT_COMMIT})",
        //     returnStdout: true
        // )

        ARTIFACT_REPOSITORY = "vandung.azurecr.io"
        CREDENTIAL_ID = "acr"

        // BUILD_IMAGE = "vandung:${GIT_COMMIT_DATE.take(10)}_${GIT_COMMIT_SHORT}"
        BUILD_IMAGE = "vandung.azurecr.io/w3/fe:${GIT_COMMIT}"
        // DEPLOY_IMAGE = "${ARTIFACT_REPOSITORY}/${BUILD_IMAGE}"
    }

    stages {
        stage ("Validate") {
            steps {
                sh "docker version"
                sh "node --version"
                sh "npm version"
                sh "kubectl get namespaces"
            }
        }

        stage ("Build docker image FE nodejs") {
            steps {
                sh "docker build -t ${BUILD_IMAGE} ."
            }
        }

        stage ("Push docker image to ACR") {
            steps {
                withDockerRegistry([credentialsId: "${CREDENTIAL_ID}", url: "http://${ARTIFACT_REPOSITORY}"]) {
                    sh "docker push ${BUILD_IMAGE}"
                }
            }
        }

        stage ("Deploy docker image to AKS") {
            steps {
                sh "kubectl set image deployment/angular-frontend angular-frontend=${BUILD_IMAGE}"
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace'
            sh "docker rmi ${BUILD_IMAGE} || true"
            deleteDir() 
        }
    }
}