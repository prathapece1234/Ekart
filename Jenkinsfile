pipeline {
    agent {label 'slave2'}

    tools {
        jdk 'jdk1.0'
        maven 'maven1.0'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        
        stage('Compile') {
            steps {
                
                sh "mvn clean compile"
            
            }
        }
        
        
        stage('Sonarq analysis') {
            steps {
                
               withSonarQubeEnv('sonar') {
                   sh '$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=shoppingcart -Dsonar.projectKey=shoppingcart -Dsonar.java.binaries=.'
                }

            }
        }
        
        
        stage('Build') {
            steps {
                
               sh 'mvn clean package -DskipTests=true'
            
            }
        }
        
        
        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker2.0') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart prathap110/shopping-cart:v2"
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker2.0') {
                        sh "docker push prathap110/shopping-cart:v2"
                    }
                }
            }
        }
        
        stage('Delete old Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker2.0') {
                        sh "docker stop shopping"
                        sh "docker rm -f shopping"
                    }
                }
            }
        }
        
        stage('Deploy To Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred1', toolName: 'docker2.0') {
                        sh "docker run --init -d --name shopping -p 8050:8070 prathap110/shopping-cart:v2"
                    }
                }
            }
        }
    }
    
    post {
        always {
            emailext body: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS: Check console output at $BUILD_URL to view the results.', 
            subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', 
            to: 'prathapece1234@gmail.com'
        }
    }
}
