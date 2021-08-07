pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }

        stage('Build-Docker-Image') {
            when {
                branch 'master' 
            }
            steps {
                script {
                    app = docker.build("rabbai/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }

        stage ('Push-Docker-Image-to-DockerHub'){
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login'){
                        app.push("${env.BUILD_NUMBER}")
                        app.push('latest')
                    }
                }
            }
        }

        stage ('Pull-trainSchedule-image-from-DockerHub-and-deploy-to-production-server') {
            steps {
                input 'Deploy to production'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p ${USERPASS} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${prod_ip} \"docker pull rabbai/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p ${USERPASS} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${prod_ip} \"docker stop rabbai/train-schedule\""
                            sh "sshpass -p ${USERPASS} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${prod_ip} \"docker rm rabbai/train-schedule\""
                        } catch (err){
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p ${USERPASS} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${prod_ip} \"docker run --restart always -d rabbai/train-schedule:${env.BUILD_NUMBER} -p 8080:8080 --name train-schedule\""
                    }
                }
                
            }
        }
    }
}
