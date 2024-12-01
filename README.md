# Flask Application with Jenkins CI/CD

This is a simple Flask application with automated deployment using Jenkins CI/CD pipeline.

## Prerequisites

- Python 3.9+
- Docker
- Jenkins
- Git

## Local Development

1. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Run the application:
```bash
python app.py
```

## Jenkins Setup

1. Install Jenkins on your Kali machine:
```bash
# Update package list
sudo apt update

# Install Java
sudo apt install openjdk-11-jdk

# Add Jenkins repository
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/sources.list.d/jenkins.list'

# Install Jenkins
sudo apt update
sudo apt install jenkins

# Start Jenkins service
sudo systemctl start jenkins
```

2. Access Jenkins at `http://localhost:8080`

3. Configure Jenkins Pipeline:
   - Create a new Pipeline job
   - Configure GitHub webhook
   - Point to the Jenkinsfile in the repository

## GitHub Setup

1. Create a new repository
2. Add this project to the repository:
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-repo-url>
git push -u origin main
```

3. Configure webhook:
   - Go to repository settings
   - Add webhook
   - Payload URL: `http://<your-jenkins-url>/github-webhook/`
   - Content type: `application/json`
   - Select: Just the push event

## Deployment

The application will be automatically deployed when you push to the main branch. The Jenkins pipeline will:
1. Check out the code
2. Install dependencies
3. Run tests
4. Build Docker image
5. Deploy the application

Access the application at `http://<your-server-ip>:5000`
