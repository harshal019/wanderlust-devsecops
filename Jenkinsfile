pipeline {
    
    agent {label 'Node'}

   
    environment {
        SCANNER_HOME=tool('sonar-scanner')
        DOCKERHUB_USERNAME= "harshalg01"

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
                    docker build -t ${DOCKERHUB_USERNAME}/wanderlust-api:${params.VERSION}  ./backend
                    docker build -t  ${DOCKERHUB_USERNAME}/wanderlust-web:${params.VERSION} ./frontend
                """
            }
        }

         stage('Trivy Image Scan') {
            steps {
                sh """
                trivy image --format table -o trivy-backend-image-report.html \
                    ${DOCKERHUB_USERNAME}/wanderlust-api:${params.VERSION}

                trivy image --format table -o trivy-frontend-image-report.html \
                    ${DOCKERHUB_USERNAME}/wanderlust-web:${params.VERSION}
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
                    docker push ${DOCKERHUB_USERNAME}/wanderlust-api:${params.VERSION}
                    docker push ${DOCKERHUB_USERNAME}/wanderlust-web:${params.VERSION}
                    docker logout                         
                    """
                }
            }
        }

       stage('Trigger CD Pipeline') {
            steps {
                build job: 'Wanderlust-CD',
                parameters: [
                    string(name: 'VERSION', value: "${params.VERSION}")
                ]
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