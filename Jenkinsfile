pipeline {
    agent any
    
    environment {
    AWS_ACCESS_KEY_ID = "your access key here"
    AWS_SECRET_ACCESS_KEY = "your secret key here"
    AWS_REGION = "region name"
    AWS_ACCOUNT_ID = "account Id"
    IMAGE_REPO_NAME = "ECR repository name"
    CLUSTER_NAME = "your cluster name"
    // ENV = "dev"
    // VERSION = "patch"
    // IMAGE_TAG = "latest"
    REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    // RELEASE_NOTES = "$GIT_COMMIT"
    // RELEASE_NOTES = sh (script: """git log -- format="medium" -1 ${GIT_COMMIT}""", returnStdout:true)
    // tag = sh(returnStdout: true, script: "git rev-parse -short=10 HEAD | tail -n +2")
     tag = sh(returnStdout: true, script: "git rev-parse --short=10 HEAD").trim()
    // tag = sh(returnStdout: true, script: "git describe --abbrev=0 --tags 2>/dev/null").trim()
    
}
        
    stages {
        stage('login into ECR') {
            steps {
                script {
                     sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                }
            }
        }
        stage('git clone') {
            steps {
                script {
                    checkout scmGit(branches: [[name: '*/{BRANCH_NAME}']], extensions: [], userRemoteConfigs: [[credentialsId: '{CREDENTIONALID}', url: '{GIT_REPO_URL}']])
                }
            }
        }
        // stage('git tagging') {
        //     steps {
        //         script {
        //             sh "chmod +x -R build/git_update.sh"
        //             sh "./build/git_update.sh -v ${VERSION} ${ENV}"
        //         }
        //     }
        // }
        stage('Docker build images') {
            steps {
                script {
                     dockerImage = docker.build "${IMAGE_REPO_NAME}:${tag}"
                }
            }
        }
        stage('Push Image to ECR') {
            steps {
                script {
                    sh "docker tag ${IMAGE_REPO_NAME}:${tag} ${REPOSITORY_URI}:${tag}"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${tag}"
                }
            }
        }
        stage('Deploy image to EKS') {
            steps {
                script {
                    sh "aws eks update-kubeconfig --name {CLUSTER_NAME} --region {AWS_REGION}"
                    sh "kubectl apply -f deployment.yaml"
                }
            }
        }
    }
}
