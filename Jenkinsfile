pipeline {
    agent any

    environment {
        DOCKER_ID = "jawi95"                      // your Docker Hub username
        DOCKER_IMAGE_MOVIE = "movieapi"           // repo name on Docker Hub
        DOCKER_IMAGE_CAST = "castapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"            // build-specific tag
        PATH = "/usr/local/bin:/usr/bin:/bin"
    }

    stages {

        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker rm -f ci-movie-api || true
                        docker rm -f ci-cast-api || true

                        echo "🔧 Building Docker images..."
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service
                    '''
                }
            }
        }

        stage('Docker Run (Smoke Test)') {
            steps {
                script {
                    sh '''
                        echo "🚀 Running containers for smoke test..."
                        docker run -d -p 8005:8000 --name ci-movie-api $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                        docker run -d -p 8006:8000 --name ci-cast-api $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                        sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        echo "🔍 Testing API Containers..."
                        # curl -f localhost:8005 
                        # curl -f localhost:8006
                    '''
                }
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials('DOCKER_HUB_PASS') // your Jenkins secret
            }
            steps {
                script {
                    sh '''
                        echo "📦 Pushing images to Docker Hub..."
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                        docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy to Dev') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh '''
                        echo "🚢 Deploying to Dev environment..."
                        rm -rf .kube && mkdir .kube
                        cat $KUBECONFIG > .kube/config

                        cp my_movie_app/values.yaml values.yml
                        sed -i "s|tag:.*|tag: \\"${DOCKER_TAG}\\"|g" values.yml

                        helm upgrade --install movieapp my_movie_app --values=values.yml --namespace dev --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh '''
                        echo "🚢 Deploying to Staging environment..."
                        rm -rf .kube && mkdir .kube
                        cat $KUBECONFIG > .kube/config

                        cp my-movie-app/values.yaml values.yml
                        sed -i "s|tag:.*|tag: \\"${DOCKER_TAG}\\"|g" values.yml

                        helm upgrade --install movieapp my_movie_app --values=values.yml --namespace staging --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to Prod') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Yes'
                }
                script {
                    sh '''
                        echo "🚢 Deploying to Production environment..."
                        rm -rf .kube && mkdir .kube
                        cat $KUBECONFIG > .kube/config

                        cp my_movie_app/values.yaml values.yml
                        sed -i "s|tag:.*|tag: \\"${DOCKER_TAG}\\"|g" values.yml

                        helm upgrade --install movieapp my_movie_app --values=values.yml --namespace prod --create-namespace
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Build failed — sending notification..."
            mail to: "mahmud.abdeljawad@gmail.com",
                 subject: "${env.JOB_NAME} - Build #${env.BUILD_ID} Failed",
                 body: "Check console output: ${env.BUILD_URL}"
        }
    }
}
