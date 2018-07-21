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
        stage('Build Docker image'){
            when{
                branch 'master'
            }
            steps{
                script {
                    app = docker.build("persistence911/train")
                    app.inside{
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push to Docker hub'){
                 when{
                    branch 'master'
                }
           
                steps{
                    script{
                        docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login'){
                            app.push("${env.BUILD_NUMBER}")
                            app.push('latest')
                        }
                    }
                }
            }

            stage('DeployToProduction'){
                when{
                    branch 'master'
                }
                steps{
                    input  'Deploy to productuion'
                    milestone(1)
                    withCredentials([usernamePassword(credentialsId: 'webserver_login' , usernameVariable: 'USERNAME' , passwordVariable: 'USERPASS')]){
                        script{
                            sh "ssshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull persistence911/train:${env.BUILD_NUMBER}\""
                            try{
                                  sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train\""
                                  sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train\""   
                            }catch(err){
                                echo "error:  $err"
                            }
                             sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always -p 8080:8080 -d  persistence911/train\""
                        }
                    }
                }
            }
        }
    }