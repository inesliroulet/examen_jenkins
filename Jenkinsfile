pipeline {
    agent any

    stages {

        stage('Configure Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBE_CONFIG')]) {
                    script {
                        sh """
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBE_CONFIG > ~/.kube/config
                        """
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh """
                    docker build -t inesliroulet/movie-service-jenkins-exam:${env.BUILD_NUMBER} ./app_files/movie-service
                    docker build -t inesliroulet/cast-service-jenkins-exam:${env.BUILD_NUMBER} ./app_files/cast-service
                    """
                }
            }
        }

        stage('Push') {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DOCKER_HUB_PASS')]) {
                    script {
                        sh """
                        echo "${DOCKER_HUB_PASS}" | docker login -u inesliroulet --password-stdin
                        docker push inesliroulet/movie-service-jenkins-exam:${env.BUILD_NUMBER}
                        docker push inesliroulet/cast-service-jenkins-exam:${env.BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Deploy to dev') {
            steps {
                script {
                    sh """
                    helm upgrade --install movie-db ./charts/charts-movie-db -n dev
                    helm upgrade --install cast-db ./charts/charts-cast-db -n dev
                    helm upgrade --install movie-service ./charts/charts-movie-service -n dev --set image.repository=inesliroulet/movie-service-jenkins-exam,image.tag=${env.BUILD_NUMBER}
                    helm upgrade --install cast-service ./charts/charts-cast-service -n dev --set image.repository=inesliroulet/cast-service-jenkins-exam,image.tag=${env.BUILD_NUMBER}
                    helm upgrade --install nginx ./charts/charts-nginx -n dev --set service.nodePort=30085
                    """
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    sh """
                    helm upgrade --install movie-db ./charts/charts-movie-db -n qa
                    helm upgrade --install cast-db ./charts/charts-cast-db -n qa
                    helm upgrade --install movie-service ./charts/charts-movie-service -n qa --set image.repository=inesliroulet/movie-service-jenkins-exam,image.tag=${env.BUILD_NUMBER}
                    helm upgrade --install cast-service ./charts/charts-cast-service -n qa --set image.repository=inesliroulet/cast-service-jenkins-exam,image.tag=${env.BUILD_NUMBER}
                    helm upgrade --install nginx ./charts/charts-nginx -n qa --set service.nodePort=30086
                    """
                }
            }
        }

        stage('Deploy to staging') {
            steps {
                script {
                    sh """
                    helm upgrade --install movie-db ./charts/charts-movie-db -n staging
                    helm upgrade --install cast-db ./charts/charts-cast-db -n staging
                    helm upgrade --install movie-service ./charts/charts-movie-service -n staging --set image.repository=inesliroulet/movie-service-jenkins-exam,image.tag=${env.BUILD_NUMBER}
                    helm upgrade --install cast-service ./charts/charts-cast-service -n staging --set image.repository=inesliroulet/cast-service-jenkins-exam,image.tag=${env.BUILD_NUMBER}
                    helm upgrade --install nginx ./charts/charts-nginx -n staging --set service.nodePort=30087
                    """
                }
            }
        }

        stage('Deploy to prod') {
            when {
                branch 'master'
            }
            steps {
                timeout(time: 5, unit: "MINUTES") {
                        input message: 'Deploy to prod ? (y/n)', ok: 'y'
                    }
                script {
                    sh """
                    helm upgrade --install movie-db ./charts/charts-movie-db -n prod
                    helm upgrade --install cast-db ./charts/charts-cast-db -n prod
                    helm upgrade --install movie-service ./charts/charts-movie-service -n prod --set image.repository=inesliroulet/movie-service-jenkins-exam,image.tag=${env.BUILD_NUMBER}
                    helm upgrade --install cast-service ./charts/charts-cast-service -n prod --set image.repository=inesliroulet/cast-service-jenkins-exam,image.tag=${env.BUILD_NUMBER}
                    helm upgrade --install nginx ./charts/charts-nginx -n prod
                    """
                }
            }
        }
    }
}
