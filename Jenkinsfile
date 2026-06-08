pipeline {
    agent any

    stages {

        stage('Get Source') {
            steps {
                git url: 'https://github.com/vgip88/pedelogo-catalogo.git', branch: 'main'
            }
        }

        stage('Docker Build Image') {
            steps {
                script {
                    dockerapp = docker.build("vgip88/api-produto:${env.BUILD_ID}",
                      '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                // 1. Atualiza a tag da imagem dinamicamente dentro do manifesto yaml
                sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api.yaml'
                sh 'cat ./k8s/api.yaml'
                
                // 2. Injeta com segurança o arquivo kubeconfig da DigitalOcean na VM do Jenkins
                // OBS: Certifique-se de que o ID aqui seja exatamente o mesmo que você criou no Secret File (ex: 'k8s-kubeconfig')
                withKubeConfig([credentialsId: 'k8s-kubeconfig']) {
                    
                    // 3. Executa o deploy aplicando os manifestos da pasta k8s direto na nuvem
                    sh 'kubectl apply -f ./k8s/api.yaml'
                    
                    // 4. Boa prática: Força o Kubernetes a atualizar os pods com a nova imagem buildada
                    sh 'kubectl rollout restart deployment/api-deployment'
                }
            }
        }
    }
}