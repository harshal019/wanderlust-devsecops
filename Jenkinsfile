pipeline {
    agent any
   
    environment {
        SCANNER_HOME=tool('sonar-scanner')
        DOCKERHUB_USERNAME= "harshalg01"
        VERSION = "v${BUILD_NUMBER}"

    }

    parameters {
        string(name: 'VERSION', defaultValue: 'v1', description: 'Docker Image Version')
    }

    stages {
        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/harshal019/wanderlust-devsecops.git'

        }
        }

        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=wanderlust \
                        -Dsonar.projectKey=wanderlust
                    '''
                }
            }
        }
        stage("Code Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }


       stage('OWASP FS SCAN') {
            steps {
                dependencyCheck(
                    additionalArguments: '--scan ./ --out .',
                    odcInstallation: 'DP-Check'
                )
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }


        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."

            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                docker build -t ${DOCKERHUB_USERNAME}/wanderlust-api:${VERSION}  ./backend
                docker build -t ${DOCKERHUB_USERNAME}/wanderlust-web:${VERSION} ./frontend
                """
            }
        }

         stage('Trivy Image Scan') {
            steps {
                sh """
                trivy image --format table -o trivy-backend-image-report.html \
                    ${DOCKERHUB_USERNAME}/wanderlust-api:${VERSION}

                trivy image --format table -o trivy-frontend-image-report.html \
                    ${DOCKERHUB_USERNAME}/wanderlust-web:${VERSION}
                """
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${DOCKERHUB_USERNAME}/wanderlust-api:${VERSION}
                    docker push ${DOCKERHUB_USERNAME}/wanderlust-web:${VERSION}
                    docker logout                         
                    """
                }
            }
        }

       stage('Deploy to Container') {                  
            steps {
                sh """
                docker compose down                       
                docker compose pull                       
                docker compose up -d                      
                """
            }
        }
    }
    post {
    always {
        emailext attachLog: true,
            subject: "${currentBuild.result}: Job ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                <html>
                <body>
                    <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                    </div>
                    <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                    </div>
                    <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                    </div>
                </body>
                </html>
            """,
            to: 'harshaldgharat01@gmail.com',
            mimeType: 'text/html',
            attachmentsPattern: 'trivy*.html'

        }
    }
}