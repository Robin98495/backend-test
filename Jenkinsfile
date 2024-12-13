pipeline {
    agent any
    // environment {
    // SONAR_TOKEN = credentials('sqp_7f124a2ffd0c4621f551dbbabd5debfda324220d')
    // DOCKER_REGISTRY = 'http://localhost:8081/' 
    // DOCKER_CREDENTIALS = credentials('sqp_7f124a2ffd0c4621f551dbbabd5debfda324220d')
    // } 
stages{
        stage("Instalación de dependencias"){
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            stages{
                stage ("Build - instalación dependencias"){
                    steps{
                        sh 'npm install'
                    }
                }
                stage("Testing"){
                    steps{
                        sh 'npm run test'
                    }
                }
                stage("Build del proyecto"){
                    steps{
                        sh 'npm run build'
                    }
                }
            }            
        }
        stage("Quality Assurance"){
            agent {
                docker {
                    label 'contenedores'
                    image 'sonarsource/sonar-scanner-cli'
                    args '--network=devops-infra_default'
                    reuseNode true
                }
            }
            stages{
                stage('Quality assurance - Paso Sonarqube (Scanner)') {
                    steps { withSonarQubeEnv('sonarqube') {
                        sh 'sonar-scanner' 
                        } 
                    } 
                }
                stage("Quality assurance - Paso puerta de calidad"){
                    steps{
                        script{
                            timeout(time: 10, unit: 'MINUTES'){
                                def qg = waitForQualityGate()
                                if (qg.status != 'OK'){
                                    error "Pipeline abortado debido a falla de puerta de calidad: ${qg.status}"
                                }
                            }                            
                        }
                    }
                }
            }
        }
        stage("Imagen docker nexus"){
            steps{
                script(){
                    docker.withRegistry("http://localhost:8081", "registry"){
                        // sh 'docker build -t backend-test-robinson .'
                        // sh 'docker tag backend-test localhost:8081/backend-test-robinson'
                        // sh 'docker tag backend-test localhost:8081/backend-test-robinson'

                        sh 'docker build -t backend-devops .'
                        sh 'docker tag backend-test:latest localhost:8082/backend-devops:latest'
                        sh 'docker push localhost:8082/backend-devops:latest'
                    }
                }                
            }
        }
    }
}
