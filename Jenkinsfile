
node {
    def imageName = 'mosubehi/customer-portal'

    try {
        stage('Checkout') {
            checkout scm

            env.IMAGE_TAG = sh(
                script: 'git rev-parse --short=12 HEAD',
                returnStdout: true
            ).trim()

            echo "Image tag: ${env.IMAGE_TAG}"
        }

        stage('Build Image') {
            sh """
                docker build -t ${imageName}:${env.IMAGE_TAG} .
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

        stage('Deploy to Kubernetes') {
            withCredentials([
                file(
                    credentialsId: 'kubeconfig-dev',
                    variable: 'KUBECONFIG_FILE'
                )
            ]) {
                withEnv(["KUBECONFIG=${env.KUBECONFIG_FILE}"]) {
                    // Update the Deployment to use the new image.
                    sh """
                        kubectl set image \
                          deployment/customer-portal \
                          customer-portal=${imageName}:${env.IMAGE_TAG} \
                          -n dev
                    """

                    try {
                        sh '''
                            set -eu

                            kubectl rollout status \
                              deployment/customer-portal \
                              -n dev --timeout=300s

                            # Test the health endpoint through port-forwarding.
                            kubectl port-forward \
                              -n dev service/customer-portal \
                              18081:5000 \
                              >/tmp/customer-portal-port-forward.log 2>&1 &
                            PF_PID=$!
                            trap 'kill "$PF_PID" 2>/dev/null || true' EXIT

                            for attempt in $(seq 1 15); do
                                if curl --silent --show-error --fail \
                                  http://127.0.0.1:18081/health \
                                  -o /tmp/customer-portal-health.json; then
                                    if grep -q '"healthy"' \
                                      /tmp/customer-portal-health.json; then
                                        cat /tmp/customer-portal-health.json
                                        echo "Smoke test passed"
                                        exit 0
                                    fi
                                fi
                                sleep 2
                            done

                            cat /tmp/customer-portal-port-forward.log
                            echo "Smoke test failed"
                            exit 1
                        '''
                    } catch (Exception deployError) {
                        echo "Deployment verification failed; attempting rollback."

                        sh '''
                            kubectl rollout undo \
                              deployment/customer-portal -n dev || true

                            kubectl rollout status \
                              deployment/customer-portal \
                              -n dev --timeout=300s || true
                        '''

                        throw deployError
                    }
                }
            }
        }

        echo "Pipeline completed successfully: ${imageName}:${env.IMAGE_TAG}"

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
