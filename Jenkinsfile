// pipeline{
//     agent any
//     option{
//         disableConcurrentBuilds()
//     }
//     environment{
//         IMAGE_NAME = "sourodip290301/multibranch-flask-app"
//         GIT_USER = "sourodip2001"
//         GIT_EMAIL = "sourodip2421@gmail"
//     }
//     stages{
//         stage("checkout"){
//             steps{
//                 checkout scm
//             }
//         }
//         stage("build & push image"){
//             when{ branch "main"}
//             steps{
//                 script{
//                 def IMAGE_TAG = "build-${BUILD_NUMBER}"
//                 withCredential([userNamePassword(credentialsId:"dockerhub-creds",usernameVariable:"DOCKER_USER",passwordVariable:"DOCKER_PASS")]){
//                     sh"docker build -t ${IMAGE_NAME}:${IMAGE_TAG}"
//                     sh"docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
//                     echo "docker login "
//                     sh"docker push ${IMAGE_NAME}:${IMAGE_TAG}"
//                 }
//                 env.IMAGE_TAG =IMAGE_TAG
//             }

//         }
//         stage("k8s deployment"){
//             when{branch 'main'}{
//                 steps{
//                     script{
//                         withCredential([userNamePassword(credentialsId:"github-creds",usernameVariable:"GIT_USER",passwordVariable:"GIT_PASS")]){
//                             sh '''
//                             set -e 
//                             git config user.name "${GIT_USER}"
//                             git config user.email "${GIT_EMAIL}"
//                             git fetch origin
//                             git checkout main
//                             git reset --hard origin/main
//                             sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml
//                             git add k8s/deployment.yml
//                             git diff --cached --quiet || git commit -m "Image tag updated to ${IMAGE_TAG}"
//                             git push https://${GIT_USER}:${GIT_PASS}@github.com/sourodip290301/prod-cicd-deployment.git main
//                              '''
//                         }
//                     }
//                 }
//             }

//             }
//         }
//     }
// }

pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "sourodip290301/multibranch-flask-app"
        GIT_EMAIL  = "sourodip2421@gmail.com"
    }

    stages {

        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Build & Push Image") {

            when {
                branch "main"
            }

            steps {
                script {

                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([
                        usernamePassword(
                            credentialsId: "dockerhub-creds",
                            usernameVariable: "DOCKER_USER",
                            passwordVariable: "DOCKER_PASS"
                        )
                    ]) {

                        sh """
                            docker build \
                                -t ${IMAGE_NAME}:${IMAGE_TAG} .

                            echo "\$DOCKER_PASS" | docker login \
                                -u "\$DOCKER_USER" \
                                --password-stdin

                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage("K8s Deployment") {

            when {
                branch "main"
            }

            steps {
                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: "github-creds",
                            usernameVariable: "GIT_USER",
                            passwordVariable: "GIT_PASS"
                        )
                    ]) {

                        sh '''
                            set -e

                            git config user.name "${GIT_USER}"
                            git config user.email "${GIT_EMAIL}"

                            git fetch origin
                            git checkout main
                            git reset --hard origin/main

                            sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml

                            git add k8s/deployment.yml

                            git diff --cached --quiet || \
                                git commit -m "Image tag updated to ${IMAGE_TAG}"

                            git push https://${GIT_USER}:${GIT_PASS}@github.com/sourodip290301/prod-cicd-deployment.git main
                        '''
                    }
                }
            }
        }
    }
}