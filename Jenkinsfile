pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '30'))
        timestamps()
        disableConcurrentBuilds()
    }
    agent {
        docker {
            label 'd3-build-agent'
            image 'openjdk:8-jdk-slim'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v /tmp:/tmp'
        }
    }
    stages {
        stage('Build') {
            steps {
                script {
                    sh "./gradlew clean build"
                }
            }
        }
        stage('Upload artifacts') {
            steps {
                script {
                    dockerTags = ['master': 'latest', 'develop': 'dev']
                    if (env.GIT_BRANCH in dockerTags || env.TAG_NAME) {
                        withCredentials([usernamePassword(credentialsId: 'nexus-d3-docker', usernameVariable: 'DOCKER_REGISTRY_USERNAME', passwordVariable: 'DOCKER_REGISTRY_PASSWORD')]) {
                            env.DOCKER_REGISTRY_URL = "https://nexus.iroha.tech:19002"
                            env.DOCKER_TAG = env.TAG_NAME ? env.TAG_NAME : dockerTags[env.GIT_BRANCH]
                            sh "./gradlew dockerPush"
                        }
                    }
                }
            }
        }
    }
    post {
        cleanup {
            cleanWs()
        }
    }
}