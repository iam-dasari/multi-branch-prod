pipeline {
    agent any

    options {
        disableConcurrentBuilds() # to avoid crashes of deployment manifest file
    }

    environment {
        IMAGE_NAME = "dasaridevops/multibranch-flask-app"
        GIT_USER = "DasariBullaiah"
        GIT_EMAIL = "dasaridevops2025@gmail.com"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Image') {
            when {
                branch 'master'
            }

            steps {
                def IMAGE_TAG = "build-${BUILD_NUMBER}"
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """

                }       
                env.IMAGE_TAG = IMAGE_TAG
            }

        }

        stage('Update K8s Manifest') {
            when { branch 'master' }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                        set -e
                        git config user.name "$GIT_USER"
                        git config user.email "$GIT_EMAIL"
                        git fetch origin
                        git checkout master
                        git reset --hard origin/master
                        sed -i 's|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                        git add k8s/deployment.yaml
                        git diff --cached --quiet || git commit -m "Updated to ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/iam-dasari/multi-branch-prod.git master
                        """
                    }
                }
            }
        }
    }
}