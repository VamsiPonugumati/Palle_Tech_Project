pipeline {

    agent any

    environment {

        IMAGE_NAME = "vamsiponugumati19/palletech"
        IMAGE_TAG = "${BUILD_NUMBER}"

        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "ekswithvamsi"

        NAMESPACE = "default"
        DEPLOYMENT_NAME = "palletech-app"
        CONTAINER_NAME = "palletech-container"
    }


    stages {


        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/VamsiPonugumati/Palle_Tech_Project.git'
            }
        }



        stage('Build Application') {

            steps {

                sh '''
                mvn clean package
                '''
            }
        }



        stage('Build Docker Image') {

            steps {

                sh '''

                docker build \
                -t $IMAGE_NAME:$IMAGE_TAG .

                docker tag \
                $IMAGE_NAME:$IMAGE_TAG \
                $IMAGE_NAME:latest

                '''
            }
        }



        stage('Docker Login') {

            steps {

                withCredentials([

                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )

                ]) {


                    sh '''

                    echo $DOCKER_PASS | docker login \
                    -u $DOCKER_USER \
                    --password-stdin

                    '''
                }
            }
        }



        stage('Push Docker Image') {

            steps {

                sh '''

                docker push $IMAGE_NAME:$IMAGE_TAG

                docker push $IMAGE_NAME:latest

                '''
            }
        }



        stage('Configure EKS') {

            steps {

                sh '''

                aws eks update-kubeconfig \
                --region $AWS_REGION \
                --name $EKS_CLUSTER


                kubectl get nodes

                '''
            }
        }




        stage('Deploy Kubernetes Manifests') {

            steps {

                sh '''

                kubectl apply \
                -f k8s/deployment.yaml \
                -n $NAMESPACE


                kubectl apply \
                -f k8s/service.yaml \
                -n $NAMESPACE

                '''
            }
        }




        stage('Update Application Image') {

            steps {

                sh '''

                kubectl set image deployment/$DEPLOYMENT_NAME \
                $CONTAINER_NAME=$IMAGE_NAME:$IMAGE_TAG \
                -n $NAMESPACE


                kubectl rollout status \
                deployment/$DEPLOYMENT_NAME \
                -n $NAMESPACE

                '''
            }
        }




        stage('Verify Deployment') {

            steps {

                sh '''

                echo "Pods"

                kubectl get pods \
                -n $NAMESPACE


                echo "Services"

                kubectl get svc \
                -n $NAMESPACE


                '''
            }
        }

    }



    post {


        success {

            echo "Application deployed successfully on EKS"

        }


        failure {

            echo "Deployment failed"

        }


        always {

            sh '''

            docker logout || true

            '''

        }

    }

}
