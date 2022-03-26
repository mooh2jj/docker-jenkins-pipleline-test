pipeline{
    agent any

    stages {

        stage('Prepare') {
            agent any

            steps {
                git credentialsId: 'git-creds', url: 'https://github.com/mooh2jj/docker-jenkins-pipleline-test.git'
            }

            post {

                success {
                    echo 'prepare success'
                }

                always {
                    echo 'done prepare'
                }

                cleanup {
                    echo 'after all other post conditions'
                }
            }
        }

        stage('Build Gradle') {
            steps{
                sh 'chmod +x gradlew'
                sh  './gradlew clean build'

                sh 'ls -al ./build'
            }
        }
        stage('Docker build image'){
            steps{
                sh 'docker build . -t mooh2jj/docker-jenkins-pipeline-test'
            }
        }
        stage('Docker push image') {
            steps {
                withCredentials([string(credentialsId: 'docker-pwd', variable: 'dockerHubPwd')]) {
                    sh "docker login -u mooh2jj -p ${dockerHubPwd}"
                }
                sh 'docker push mooh2jj/docker-jenkins-pipeline-test'
            }

            post {
                success {
                    echo 'success'
                }

                failure {
                    echo 'failed'
                }
            }
        }
        stage('Run Container on SSH Dev Server'){
            steps{
                echo 'SSH'
                sshagent (credentials: ['ubuntu-server2']) {
//                     sh 'ssh -vvv -o StrictHostKeyChecking=no -T ubuntu@172.31.40.133 "whoami"'
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.40.133 'docker run -p 8080:8080 -d mooh2jj/docker-jenkins-pipeline-test'"
                }

            }

        }

    }
}