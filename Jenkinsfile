pipeline {
    agent {
        docker {
            image 'abhishekf5/maven-abhishek-docker-agent:v1'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // Mount Docker socket
        }
    }

    environment {
        SONAR_URL = "http://15.206.82.204:9000/"  // Update with your SonarQube server IP
        DOCKER_IMAGE = "bhargavibindu/ultimate-cicd-pipeline:${BUILD_NUMBER}" // Update with your DockerHub username
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                sh 'cd spring-boot-app && mvn clean package'
            }
        }

        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                    cd spring-boot-app
                    mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}
                    '''
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh 'cd spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-creds') {
                    dockerImage.push()
                    }
                }
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "https://github.com/bhargavi-bindu/Jenkins_CI-CD_Pipeline.git"
                GIT_USER_NAME = "bhargavi-bindu"
            }
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                    git config --global user.email "bhargavi.bindu2000@gmail.com"
                    git config --global user.name "bhargavi-bindu"
                    git checkout main || git checkout -b main
                    git pull origin main
                    sed -i "s#\\(image: bhargavibindu/ultimate-cicd-pipeline:\\).*#\\1${BUILD_NUMBER}#" spring-boot-app-manifests/deployment.yml
                    cat spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}" || echo "No changes to commit"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
} // End of pipeline

// 📌 **Things to Update:**
// - `<sonar-cred-id>` → Your SonarQube credential ID in Jenkins
// - `<docker-cred-id>` → Your DockerHub credential ID in Jenkins
// - `<your-dockerhub-username>` → Your DockerHub username
// - `<your-repo-name>` → Your GitHub repository name
// - `<your-github-username>` → Your GitHub username
// - `<your-email>` → Your GitHub email

// Let me know if you’d like me to tweak anything or add extra stages! 🚀
