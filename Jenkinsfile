pipeline {
    agent any
    
    environment {
        SONAR_PROJECT_KEY = 'flask-app'
        GITHUB_REPO = 'https://github.com/your-username/your-repo.git'
        GITHUB_CREDENTIALS = credentials('github-credentials')
        ANSIBLE_VAULT_CREDENTIALS = credentials('ansible-vault-password')
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Clean workspace before cloning
                cleanWs()
                // Clone the repository using credentials
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: "${GITHUB_REPO}"
            }
        }
        stage('Setup Infrastructure') {
            steps {
                // Write Ansible vault password to a temporary file
                writeFile file: '.vault_pass', text: "${ANSIBLE_VAULT_CREDENTIALS}"
                
                // Run Ansible playbook
                sh '''
                    # Install Ansible if not present
                    which ansible-playbook || (apt-get update && apt-get install -y ansible)
                    
                    # Run the playbook
                    ansible-playbook -i ansible/inventory.ini ansible/playbook.yml \
                        --vault-password-file .vault_pass
                '''
                
                // Clean up vault password file
                sh 'rm -f .vault_pass'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                    python -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                    pip install pytest pytest-cov safety bandit
                '''
            }
        }
        stage('Security Checks') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh '''
                            . venv/bin/activate
                            safety check
                        '''
                    }
                }
                
                stage('Static Security Analysis') {
                    steps {
                        sh '''
                            . venv/bin/activate
                            bandit -r . -f json -o bandit-report.json
                        '''
                    }
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest --cov=. --cov-report=xml --junitxml=test-results.xml
                '''
            }
            post {
                always {
                    junit 'test-results.xml'
                    cobertura coberturaReportFile: 'coverage.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=. \
                        -Dsonar.python.coverage.reportPaths=coverage.xml \
                        -Dsonar.python.xunit.reportPath=test-results.xml \
                        -Dsonar.python.bandit.reportPaths=bandit-report.json \
                        -Dsonar.python.version=3.9
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    . venv/bin/activate
                    pip install gunicorn
                    # Kill any existing gunicorn process
                    pkill gunicorn || true
                    # Start the application with gunicorn
                    gunicorn --bind 0.0.0.0:5000 --daemon app:app
                    
                    # Verify deployment
                    sleep 5
                    curl http://localhost:5000/health || (echo "Deployment failed" && exit 1)
                '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                color: 'good',
                message: "Build Successful: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
            )
        }
    }
}
