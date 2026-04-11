@Library('Shared') _
pipeline{
    agent {label 'node'}

    environment{
        SONAR_HOME = tool "Sonar"
    }

    stages{
        stage('Workspace cleanup'){
            steps{
                script{
                    clean_ws()
                }
            }
        }

        stage('Git: Repo clone'){
            steps{
                script{
                    code_checkout("https://github.com/kaustavGitH/2048_game.git","main")
                }
            }
        }

        stage('Trivy: Filescan system'){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage('Sonarqube: Code analysis'){
            steps{
                script{
                    sonarqube_analysis("Sonar","2048_game","2048_game")
                }
            }
        }

        stage('SonarQube: Code Quality Gates'){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }

        stage('Docker: build/push'){
            steps{
                echo "docker image is build and pushed"
            }
        }

        stage('Dev deployment approval'){
            steps{
                timeout(time: 15, unit: "MINUTES"){
                    input message:'Do you want to approve the deployment?',ok:'Proceed'
                }

                echo "Initiating deployment"
            }
        }

        stage('Deploy on dev'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'devops-cluster.us-east-1.eksctl.io', contextName: '', credentialsId: 'k8s-cred', namespace: 'gameapps', restrictKubeConfigAccess: false, serverUrl: 'https://6D6C515BA875CB036EAE07E744E23910.gr7.us-east-1.eks.amazonaws.com'){
                    sh """
                        kubectl apply -f deploy.yml
                        kubectl apply -f service.yml
                        kubectl apply -f ingress.yml
                    """    
                }
            }
        }

        stage('Verify deployment'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'devops-cluster.us-east-1.eksctl.io', contextName: '', credentialsId: 'k8s-cred', namespace: 'gameapps', restrictKubeConfigAccess: false, serverUrl: 'https://6D6C515BA875CB036EAE07E744E23910.gr7.us-east-1.eks.amazonaws.com'){
                    sh """
                        kubectl get all -n gameapps
                    """
                }
            }
        }
    }
}