pipeline {
    agent any

    tools {
        jdk 'jdk'
        nodejs 'nodejs'   // <-- Change to whatever is configured in Jenkins
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        SONAR_AUTH_TOKEN = credentials('Sonar-token')
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Aseemakram19/hotstar-kubernetes.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Hotstar \
                        -Dsonar.projectName=Hotstar \
                        -Dsonar.sources=. \
                        -Dsonar.login=${SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker']) {
                        sh "docker build -t hotstar ."
                        sh "docker tag hotstar aseemakram19/hotstar:latest"
                        sh "docker push aseemakram19/hotstar:latest"
                    }
                }
            }
        }

        stage('TRIVY Image Scan') {
            steps {
                sh "trivy image aseemakram19/hotstar:latest > trivyimage.txt"
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                    docker rm -f hotstar || true
                    docker run -d --name hotstar -p 3000:3000 aseemakram19/hotstar:latest
                """
            }
        }
    }
}
