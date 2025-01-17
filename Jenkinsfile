pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh '/var/lib/jenkins/.local/bin/pytest -v'
                
            }
        }
        stage('Build_docker_image') {
            steps {
                sh 'docker build -f Dockerfile -t kotovak/app:$GIT_COMMIT .'
                
            }
        }
        stage('Push_docker_image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub_cred_jenkins', url: 'https://index.docker.io/v1/'){
                    sh '''
                        docker push kotovak/app:$GIT_COMMIT
                    '''
                }    
                
            }
        }
        stage('RunContainer on webapp server') {
            steps {
                script {
                  def dockerStop = "docker rm -f app"
                  def dockerRun = "docker run -p 80:5000 -d --name app kotovak/app:$GIT_COMMIT"
                  sshagent(['webapp-server']) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.5.134 '${dockerStop}'"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.5.134 '${dockerRun}'"
                  } 
                } 
            }               
        }
    }
}
