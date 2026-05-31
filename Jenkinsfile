pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        DOCKERHUB_CREDS = credentials('DOCKERHUB_CREDENTIALS')
        GITHUB_CREDS    = credentials('GITHUB_CREDENTIALS')
        DOCKERHUB_USER  = 'devkrishan001'
        IMAGE_TAG       = "v${BUILD_NUMBER}"
        SERVICE         = 'backend'
    }

    triggers {
        pollSCM('* * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    echo "🔨 Building ${SERVICE}:${IMAGE_TAG}..."
                    docker build -t ${DOCKERHUB_USER}/${SERVICE}:${IMAGE_TAG} .
                    docker tag ${DOCKERHUB_USER}/${SERVICE}:${IMAGE_TAG} \
                               ${DOCKERHUB_USER}/${SERVICE}:latest
                    echo "✅ Build done"
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh """
                    echo ${DOCKERHUB_CREDS_PSW} | \
                    docker login -u ${DOCKERHUB_CREDS_USR} --password-stdin
                    docker push ${DOCKERHUB_USER}/${SERVICE}:${IMAGE_TAG}
                    docker push ${DOCKERHUB_USER}/${SERVICE}:latest
                    echo "✅ Pushed ${SERVICE}:${IMAGE_TAG}"
                """
            }
        }

        stage('Update GitOps Repo') {
            steps {
                sh """
                    rm -rf gitops-environments
                    git clone https://\${GITHUB_CREDS_USR}:\${GITHUB_CREDS_PSW}@github.com/dev-krishan-dhaka/gitops-environments.git

                    cd gitops-environments

                    yq e '.image.tag = "${IMAGE_TAG}"' \
                       -i dev/user-management/backend-values.yaml

                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins CI"
                    git add dev/user-management/backend-values.yaml
                    git diff --cached --quiet || \
                        git commit -m "ci: update backend to ${IMAGE_TAG} [skip ci]"

                    for i in 1 2 3; do
                        git push && break
                        echo "retry \$i..."
                        git pull --rebase
                        sleep 5
                    done

                    cd .. && rm -rf gitops-environments
                    echo "✅ GitOps updated"
                """
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo "✅ Backend ${IMAGE_TAG} pipeline done!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
