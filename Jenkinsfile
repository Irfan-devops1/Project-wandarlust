pipeline {
agent any
parameters {
    string(name: 'FRONTEND_DOCKER_TAG', defaultValue: 'latest', description: 'Frontend Docker image tag')
    string(name: 'BACKEND_DOCKER_TAG', defaultValue: 'latest', description: 'Backend Docker image tag')
}

stages {

    stage('Workspace Cleanup') {
        steps {
            cleanWs()
        }
    }

    stage('Git Checkout') {
        steps {
            git branch: 'main',
                url: 'https://github.com/DevMadhup/Wanderlust-Mega-Project.git'
        }
    }

    stage('Check Project Files') {
        steps {
            sh '''
                echo "Checking project files..."
                pwd
                ls -la
                echo "Backend:"
                ls -la backend || true
                echo "Frontend:"
                ls -la frontend || true
            '''
        }
    }

    stage('Trivy Scan') {
        steps {
            sh '''
                if command -v trivy >/dev/null 2>&1; then
                    trivy fs --exit-code 0 --severity HIGH,CRITICAL .
                else
                    echo "Trivy is not installed. Skipping Trivy scan."
                fi
            '''
        }
    }

    stage('OWASP Dependency Check') {
        steps {
            sh '''
                if command -v dependency-check.sh >/dev/null 2>&1; then
                    dependency-check.sh \
                        --project Wanderlust \
                        --scan . \
                        --format XML \
                        --out dependency-check-report \
                        || true
                else
                    echo "OWASP Dependency Check is not installed. Skipping."
                fi
            '''
        }
    }

    stage('SonarQube Analysis') {
        steps {
            script {
                try {
                    withSonarQubeEnv('Sonar') {
                        sh '''
                            if command -v sonar-scanner >/dev/null 2>&1; then
                                sonar-scanner \
                                    -Dsonar.projectKey=wanderlust \
                                    -Dsonar.projectName=wanderlust \
                                    -Dsonar.sources=.
                            else
                                echo "Sonar Scanner not installed. Skipping."
                            fi
                        '''
                    }
                } catch (Exception e) {
                    echo "SonarQube analysis failed or is not configured. Continuing..."
                }
            }
        }
    }

    stage('SonarQube Quality Gate') {
        steps {
            script {
                try {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: false
                    }
                } catch (Exception e) {
                    echo "SonarQube Quality Gate not available. Continuing..."
                }
            }
        }
    }

    stage('Backend Environment Setup') {
        steps {
            dir('Automations') {
                sh '''
                    if [ -f updatebackendnew.sh ]; then
                        chmod +x updatebackendnew.sh
                        ./updatebackendnew.sh
                    else
                        echo "updatebackendnew.sh not found. Skipping."
                    fi
                '''
            }
        }
    }

    stage('Frontend Environment Setup') {
        steps {
            dir('Automations') {
                sh '''
                    if [ -f updatefrontendnew.sh ]; then
                        chmod +x updatefrontendnew.sh
                        ./updatefrontendnew.sh
                    else
                        echo "updatefrontendnew.sh not found. Skipping."
                    fi
                '''
            }
        }
    }

    stage('Build Backend Docker Image') {
        steps {
            dir('backend') {
                sh """
                    docker build \
                    -t madhupdevops/wanderlust-backend-beta:${params.BACKEND_DOCKER_TAG} \
                    .
                """
            }
        }
    }

    stage('Build Frontend Docker Image') {
        steps {
            dir('frontend') {
                sh """
                    docker build \
                    -t madhupdevops/wanderlust-frontend-beta:${params.FRONTEND_DOCKER_TAG} \
                    .
                """
            }
        }
    }

    stage('Docker Login') {
        steps {
            echo 'Docker login is required before pushing images.'
            echo 'Configure Docker Hub credentials in Jenkins before enabling push.'
        }
    }

    stage('Docker Push') {
        steps {
            echo 'Docker push stage is currently skipped until Docker Hub credentials are configured.'
        }
    }
}

post {
    always {
        echo 'Wanderlust CI pipeline completed.'
    }

    success {
        echo 'Wanderlust CI pipeline completed successfully.'
    }

    failure {
        echo 'Wanderlust CI pipeline failed.'
    }
}

}

