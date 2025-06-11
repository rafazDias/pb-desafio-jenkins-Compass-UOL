pipeline{
    agent any
    post {
        always {
            // Chuck Norris aparece em todos os builds
            chuckNorris()
        }
       
        success {
            echo '🚀 Deploy realizado com sucesso!'
            echo '💪 Chuck Norris aprova seu pipeline DevSecOps!'
            echo "✅ Imagem jamalshadowdev/meuapp:${env.BUILD_ID} deployada no Kubernetes"
        }
       
        failure {
            echo '❌ Build falhou, mas Chuck Norris nunca desiste!'
            echo '🔍 Chuck Norris está investigando o problema...'
            echo '💡 Verifique: Docker build, DockerHub push ou Kubernetes deploy'
        }
       
        unstable {
            echo '⚠️ Build instável - Chuck Norris está monitorando'
        }
    }
    stages{
        stage('Docker build backend') {
            steps{
                script{
                    backend = docker.build("rafaelldiass/pipedesafiojenkins:backend-${env.BUILD_ID}", '--no-cache -f ./backend/Dockerfile ./backend' )
                }
            }
        }
         stage('Docker build frontend') {
            steps{
                script{
                    frontend = docker.build("rafaelldiass/pipedesafiojenkins:frontend-${env.BUILD_ID}", '--no-cache -f ./frontend/Dockerfile ./frontend' )
                }
            }
        }
            stage('Push Image') {
            steps{
                script{
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub'){
                        backend.push('latest-backend')
                        backend.push("backend-${env.BUILD_ID}")
                        frontend.push('latest-frontend')
                        frontend.push("frontend-${env.BUILD_ID}")
                    }
                
                }
            }
        }

                    stage('Deploy Kubernetes') {
                        environment {
                            tag_version =  "${env.BUILD_ID}"
                        }
                steps{
                    withKubeConfig([credentialsId: 'kubeconfig']){
                        
                        //backend
                        sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment-backend.yaml'
                        sh "kubectl apply -f k8s/deployment.yaml"
                        
                        //frontend

                        sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment-frontend.yaml'
                        sh "kubectl apply -f k8s/deployment.yaml"
                    }
                    
                }
        }
    }
}