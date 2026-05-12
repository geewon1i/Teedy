pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'kimjiwon777/teedy'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        WSL_WORKSPACE = '/mnt/c/Users/94875/.jenkins/workspace/Teedy2'
    }

    stages {
        stage('Build') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/geewon1i/Teedy.git'
                    ]]
                )

                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Building image') {
            steps {
                bat """
                wsl -d Ubuntu -- bash -lc "cd ${WSL_WORKSPACE} && docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                """
            }
        }

        stage('Upload image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '1',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    wsl -d Ubuntu -- bash -lc "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
                    wsl -d Ubuntu -- bash -lc "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    wsl -d Ubuntu -- bash -lc "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                    wsl -d Ubuntu -- bash -lc "docker push ${DOCKER_IMAGE}:latest"
                    """
                }
            }
        }

        stage('Run containers') {
            steps {
                bat """
                wsl -d Ubuntu -- bash -lc "docker stop teedy-container-8081 || true"
                wsl -d Ubuntu -- bash -lc "docker rm teedy-container-8081 || true"
                wsl -d Ubuntu -- bash -lc "docker run --name teedy-container-8081 -d -p 8081:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                wsl -d Ubuntu -- bash -lc "docker ps --filter name=teedy-container"
                """
            }
        }
    }
}