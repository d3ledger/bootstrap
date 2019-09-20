def dockerTags = ['master': 'latest', 'develop': 'develop']
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
                    sh "./gradlew clean build --info"
                }
            }
        }
        stage('Test') {
            steps {
                 script {
                    sh "./gradlew test --info"
                 }
            }
        }
        stage('Push artifacts') {
            when {
              expression { return (env.GIT_BRANCH in dockerTags || env.TAG_NAME) }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus-d3-docker', usernameVariable: 'DOCKER_REGISTRY_USERNAME', passwordVariable: 'DOCKER_REGISTRY_PASSWORD')]) {
                        env.DOCKER_REGISTRY_URL = "https://nexus.iroha.tech:19002"
                        env.DOCKER_TAG = env.TAG_NAME ? env.TAG_NAME : dockerTags[env.GIT_BRANCH]
                        sh "./gradlew dockerPush"
                    }
                }
            }
        }
        stage('Sonar') {
                  steps {
                    script {
                      if (env.BRANCH_NAME == 'develop') {
                        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]){
                          sh(script: "./gradlew sonarqube -x test --configure-on-demand \
                            -Dsonar.links.ci=${BUILD_URL} \
                            -Dsonar.github.pullRequest=${env.CHANGE_ID} \
                            -Dsonar.github.disableInlineComments=true \
                            -Dsonar.host.url=https://sonar.soramitsu.co.jp \
                            -Dsonar.login=${SONAR_TOKEN} \
                            ")
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
