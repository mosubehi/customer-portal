node {
    def imageName = 'mosubehi/customer-portal'

    try {
        stage('Checkout') {
            checkout scm

            env.IMAGE_TAG = sh(
                script: 'git rev-parse --short=12 HEAD',
                returnStdout: true
            ).trim()

            echo "Building image tag: ${env.IMAGE_TAG}"
        }

        stage('Build Image') {
            sh """
                docker build \
                    -t ${imageName}:${env.IMAGE_TAG} \
                    .
            """
        }

        stage('Push Image') {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )
            ]) {
                sh """
                    set +x
                    echo "\$DOCKERHUB_TOKEN" | docker login \
                        --username "\$DOCKERHUB_USER" \
                        --password-stdin

                    docker push ${imageName}:${env.IMAGE_TAG}
                    docker logout
                """
            }
        }

        currentBuild.result = 'SUCCESS'
        echo "Successfully published ${imageName}:${env.IMAGE_TAG}"

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
