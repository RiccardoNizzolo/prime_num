#!/usr/bin/env groovy

/**
 * Jenkinsfile
 */
pipeline {
    agent any
    options {
        buildDiscarder(
            // Only keep the 10 most recent builds
            logRotator(numToKeepStr:'10'))
    }
    environment {
        projectName = ''
        emailTo = ''
        emailFrom = ''
        VIRTUAL_ENV = "${env.WORKSPACE}/venv"
    }

    stages {




        stage ('Build image and test') {
            steps {

                sh 'docker image build -t "pndweb:1" -f docker/Dockerfile .'
                sh 'docker image build -t "pndweb:latest" -f docker/Dockerfile .'
                sh 'docker run -it "pndweb:latest" "python manage.py test"'
            }
        }
        stage ('train') {
            steps {
                //

                sh 'docker-compose -f docker/docker-compose-model_build.yml up -d'

                timeout(20) {
                    waitUntil {
                       script {
                         def r = sh script: 'wget -q http://docker_pnd-model-builder_1:5000/models', returnStatus: true
                         return (r == 0);
                       }
                    }
                }
                sh 'docker commit docker_pnd-model-builder_1 pndwebprod:latest'
                sh 'docker container stop $( docker ps -q --filter="name=pnd-model-builder*")'

            }


        }

                stage ('deploy') {
            steps {
                //

                sh 'docker-compose -f docker/docker-compose-model_deploy.yml up -d'

                timeout(20) {
                    waitUntil {
                       script {
                         def r = sh script: 'wget -q http://docker_pnd-model-app_1:5000/models', returnStatus: true
                         return (r == 0);
                       }
                    }
                }


            }


        }


    }


}