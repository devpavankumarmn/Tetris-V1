pipeline {
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/devpavankumarmn/Tetris-V1.git'
            }
        }
        stage('Sonar analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=TETRISVersion1.0 \
                    -Dsonar.projectKey=TETRISVersion1.0 '''
                }
            }
        }
        stage('QG') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            }
        }
        stage('NPM') {
            steps {
               sh 'npm install'
            }
        }
        stage('TRIVY FS') {
            steps {
               sh 'trivy fs . > trivy.txt'
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker build and push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                     sh '''
                     docker build -t tetrisv1 .
                     docker tag tetrisv1 Pavanf5/tetrisv1:latest
                     docker push Pavanf5/trtrisv1:latest
                    } 
                }
            }
        }
        stage('TRIVY IMAGE') {
            steps {
               sh 'trivy Image Pavanf5/tetrisv1:latest > trivyimage.txt'
            }
        }
    }
}
