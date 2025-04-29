pipeline {
    agent any

    environment {
        SNYK_CREDENTIALS = 'snyk-token'
        DOCKERHUB_CREDENTIALS = 'docker-hub-credentials'
        BACKEND_HEALTH_URL = 'http://34.133.27.32:31400/data'
        ARGOCD_SERVER = '34.133.27.32:31125'  
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ylmzzeyneep/DevOpsUseCase.git'
            }
        }

        stage('Snyk Security Scan (Frontend Only)') {
            steps {
                script {
                    withCredentials([string(credentialsId: SNYK_CREDENTIALS, variable: 'SNYK_TOKEN')]) {
                        dir('frontend') {
                            sh 'snyk auth $SNYK_TOKEN'
                            sh 'snyk test || true'
                            echo "✅ Frontend için Snyk güvenlik taraması tamamlandı."
                        }
                    }
                }
            }
        }

        stage('Build Images') {
            parallel {
                stage('Build Backend Image') {
                    steps {
                        script {
                            dir('backend') {
                                dockerImage = docker.build("ylmzzeyneep/backend:${env.BUILD_NUMBER}")
                                echo "✅ Backend imajı başarıyla build edildi."
                            }
                        }
                    }
                }
                stage('Build Frontend Image') {
                    steps {
                        script {
                            dir('frontend') {
                                dockerImage = docker.build("ylmzzeyneep/frontend:${env.BUILD_NUMBER}")
                                echo "✅ Frontend imajı başarıyla build edildi."
                            }
                        }
                    }
                }
            }
        }

          stage('Trivy Image Scan (Frontend Only)') {
            steps {
                script {
                    sh "trivy image ylmzzeyneep/frontend:${env.BUILD_NUMBER} || true"
                    echo "✅ Frontend imajı için Trivy taraması tamamlandı."
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        echo "✅ Docker Hub'a başarıyla giriş yapıldı."
                    }
                }
            }
        }

        stage('Push Images') {
            parallel {
                stage('Push Backend Image') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                                dockerImage.push()
                                echo "✅ Backend imajı başarıyla Docker Hub'a yüklendi."
                            }
                        }
                    }
                }
                stage('Push Frontend Image') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                                dockerImage.push()
                                echo "✅ Frontend imajı başarıyla Docker Hub'a yüklendi."
                            }
                        }
                    }
                }
            }
        }

        stage('Test Backend Health Check') {
            steps {
                script {
                    def status = sh(script: "curl -s -o /dev/null -w '%{http_code}' ${BACKEND_HEALTH_URL}", returnStdout: true).trim()
                    if (status == "200") {
                        echo "✅ Backend servisi çalışıyor."
                    } else {
                        error "❌ Backend servisi çalışmıyor! HTTP Yanıt Kodu: ${status}"
                    }
                }
            }
        }

    }
}
