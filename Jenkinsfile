pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'python3 -m py_compile app.py'
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m pytest'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    withEnv(["PATH+SONAR=${tool 'SonarScanner'}/bin"]) {
                        sh '''
                            sonar-scanner \
                              -Dsonar.projectKey=hello-python \
                              -Dsonar.projectName=hello-python \
                              -Dsonar.sources=. \
                              -Dsonar.python.version=3
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['gce-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no sanahjhvr@34.14.195.25 "
                            mkdir -p ~/app
                            cd ~/app
                            git clone -q https://github.com/sanah28/hello-python.git . 2>/dev/null || git pull
                            python3 -m pip install --user flask pytest
                            nohup python3 app.py > app.log 2>&1 &
                        "
                    '''
                }
            }
        }
    }
}
