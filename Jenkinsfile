pipeline {
    agent any
    
    stages {     

        stage("Checkout Source"){
            steps {
                git url: 'https://github.com/yduretti/k8s_digoce', branch: 'main'
                sh """
                find /var/lib/jenkins/workspace/pedelogo/
                """   
            }
        }

        stage("Install kubectl"){
            steps {
                sh """
                    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
                    chmod +x ./kubectl
                    ./kubectl version --client
                """               
            }
        }

        stage("Build Image"){
            steps {
                sh """
                   ls -l
                """   
                script {
                    dockerapp = docker.build("yduretti/pedelogo:${env.BUILD_ID}",
                    '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage("Push Image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }                    
                }
            }
        }

        // stage("Push Kubernetes") {
        //     // agent {
        //     //     kubernetes {
        //     //         cloud 'do-kube-dev'
        //     //     }
        //     // }
        //     environment {
        //         tag_version = "${env.BUILD_ID}"
        //     }
        //     steps {
        //         // withKubeConfig([credentialsId: 'kubeconfig']) {
        //         //     sh """
        //         //         sed -i "s/{{tag}}/${tag_version}/g" ./k8s/api/deployment.yaml
        //         //         cat ./k8s/api/deployment.yaml
        //         //         kubectl apply -f ./k8s -R
        //         //     """
        //         // }  
        //         script
        //         {
        //             sh 'pwd'
        //             sh 'ls -l'
        //             //sh 'sed -i "s/{{tag}}/${tag_version}/g" ./k8s/api/deployment.yaml'
        //             //sh 'cat ./k8s/api/deployment.yaml'
        //             //kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
        //         }
        //     }          
        // }

        stage("Push Kubernetes") {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                        sed -i "s/{{tag}}/$tag_version/g" ./k8s/api/deployment.yaml
                        cat ./k8s/api/deployment.yaml
                        kubectl apply -f ./k8s -R                        
                    """
                    // ./kubectl get all
                }  
            }          
        }       
    }
}