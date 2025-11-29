pipeline {
    agent any

    tools {
        jdk 'jdk'
        nodejs 'nodejs'   // <-- Change to whatever is configured in Jenkins
    }

    environment {
        SCANNER_HOME = tool 'SonaQube Scanner'
        SONAR_AUTH_TOKEN = credentials('token')
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
                    url: 'https://github.com/techieak2410/hotstar-kubernetes-ashish.git'
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
                    waitForQualityGate abortPipeline: false, credentialsId: 'token'
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
                        sh "docker tag hotstar AshishG/hotstar:latest"
                        sh "docker push AshishG/hotstar:latest"
                    }
                }
            }
        }

        stage('TRIVY Image Scan') {
            steps {
                sh "trivy image AshishG/hotstar:latest > trivyimage.txt"
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                    docker rm -f hotstar || true
                    docker run -d --name hotstar -p 3000:3000 AshishG/hotstar:latest
                """
            }
        }
    }
}
