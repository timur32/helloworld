#!groovy

pipeline {

    agent any
    
    environment {

        host="vm01"
        registry = "timur32/helloworld"
        registryCredentials = "mydockerhub"

    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: "3"))
    }

    stages {
        stage('1-pull sources') {
            steps {
               checkout scm: [$class: "GitSCM",
               userRemoteConfigs: [[url: "git@github.com:timur32/helloworld.git", credentialsId: "mygit" ]],
               branches: [[name: "dev"]]]
            }
        }
        stage('2-build docker') {
            steps {
                script {
                    docker.withRegistry( '', registryCredentials){
                        def pythonimage = docker.build(registry + ":${BUILD_NUMBER}-python", "-f ./docker/python/Dockerfile .")
                        pythonimage.push()
                        def phpimage = docker.build(registry + ":${BUILD_NUMBER}-php", "-f ./docker/php/Dockerfile .")
                        phpimage.push()
                        def nginximage = docker.build(registry + ":${BUILD_NUMBER}-nginx", "-f ./docker/nginx/Dockerfile .")
                        nginximage.push()
                    }
                }
            }
        }
        stage('3-Clean docker images') {
            steps {
                sh "docker images -a | grep timur32 | awk '{print \$3}' | xargs docker rmi -f"
            }
        }
        stage('4-Deploy') {
            steps {
                script {
                    docker.withRegistry( '', registryCredentials){
                        sh """
                        sed -ie 's|REGISTRY:TAG|${registry}:${BUILD_NUMBER}|g' ./docker/docker-compose.yml  
                        scp ./docker/docker-compose.yml ${host}:
                        ssh ${host} '
                        hostname
                        docker-compose up --force-recreate -d
                        '
                        """
                    }
                }
            }
        }
    }
}
