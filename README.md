```
pipeline {
    agent any
    stages {
        stage ("git checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/lavanyajatla/Netflix.git'
            }
        }
        stage ("Building docker imgae") {
            steps {
                sh "docker build -t netflix:v1 ."
            }
        }
        stage ("Pushing docker image to docker hub") {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub' , url: 'https://index.docker.io/v1/') {
                    script {
                        sh "docker tag netflix:v1 lavanyajatla814/netflix:latest"
                        sh "docker push lavanyajatla814/netflix:latest"
                    }
                }
            }
        }
        stage ("Running Docker Container") {
            steps {
                sh "docker run -itd --name netflix -p 4000:80 lavanyajatla814/netflix:latest"
            }
        }
        stage('Deploy to kubernetes'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f k8s.yaml'
                  }
                }
            }
        }
    }
}
```
