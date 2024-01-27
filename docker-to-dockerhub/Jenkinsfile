pipeline {
    agent any

    tools {
        maven 'Maven-3.9.6'
        jdk 'JDK-17'
    }

    environment {
        DOCKER_IMAGE_NAME = 'hkaratas/docker-git'
        DOCKER_IMAGE_TAG = 'tagname'
    }

    stages {
        stage('github repo çekme') {
            steps {
                script {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/hkrts91/devops.git'
                }
            }
        }

        stage('Build işlemi') {
            steps {
                script {
                    // Maven komutlarını buraya ekleyin
                    bat 'mvn clean install'
                }
            }
        }

        stage('Snyk taraması') {
            steps {
                script {
                    def snykPath = 'C:\\Users\\Karatas\\AppData\\Roaming\\npm\\snyk'

                    try {
                        // Snyk taramasını gerçekleştir
                        def snykCommand = bat(script: "\"${snykPath}\" test --json", returnStatus: true).trim()

                        if (snykCommand != 0) {
                            error "Snyk scan failed. Exit code: ${snykCommand}"
                        }

                        def snykResult = readJSON text: snykCommand

                        if (snykResult.issues.length > 0) {
                            echo "Security vulnerabilities found! Please check Snyk for details."
                            snykResult.issues.each { issue ->
                                echo "Issue: ${issue.title}"
                                echo "Severity: ${issue.severity}"
                                echo "URL: ${issue.url}"
                            }
                            error "Security vulnerabilities found! See above for details."
                        } else {
                            echo "No security vulnerabilities found."
                        }
                    } catch (Exception e) {
                        // Hata durumunu görmezden gel
                        echo "Snyk Scan failed, but ignoring the error."
                    }
                }
            }
        }

        stage('Build Docker Image oluşturması') {
            steps {
                script {
                    // Belirtilen Dockerfile'ı kullanarak Docker imajını oluştur
                    bat "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                }
            }
        }

        stage('Docker-huba gönderme') {
            steps {
                script {
                    // Jenkins Credential Manager'dan Docker Hub kullanıcı adı ve şifresini al
                    withCredentials([usernamePassword(credentialsId: 'docker-git', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                        bat "docker login -u %DOCKER_HUB_USERNAME% -p %DOCKER_HUB_PASSWORD%"

                        // Docker Image'ını Docker Hub'a gönder
                        bat "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }

        stage('docker-hub repo çekme') {
            steps {
                script {
                    // Docker Hub'dan Docker imajını çek
                    bat "docker pull ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('docker-hub repo snyk taraması') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                        def snykCommand = "\"C:\\Users\\Karatas\\AppData\\Roaming\\npm\\snyk\" test --json"

                        try {
                            bat "echo 'SNYK_TOKEN=${SNYK_TOKEN}' > .env"
                            bat script: snykCommand, returnStatus: true
                        } catch (Exception e) {
                            echo "Snyk test failed, but ignoring the error."
                        }
                    }
                }
            }
        }
    }
}
