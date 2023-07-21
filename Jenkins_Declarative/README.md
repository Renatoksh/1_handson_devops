# 1_handson_devops
# Diagram
#
![](./img/Diagram.PNG)
## Get Started
## 1.  Create and configure GitHub Repository
![](./img/Github_config.PNG)
#
## 2. Create and Connect EC2 in AWS
#
## 3. Clone the repository from GitHub repository
### git clone https://github.com/Renatoksh/django-notes-app.git
![](./img/clone_repository.PNG)
#
## 4. Install Docker on the EC2 just created
```
sudo apt-get update
```
```
sudo apt install docker.io -y
```
![](./img/docker_version.PNG)
#
![](./img/docker_permissions.PNG)
#
![](./img/docker_permissions_1.PNG)
#
## 5. Create container
```
docker build -t notes-app .
```
## 6. Install Jenkins
```
sudo apt update
sudo apt install openjdk-17-jre
```
#
```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
#
![](./img/Jenkins_pass.PNG)
#
![](./img/Jenkins_startus.PNG)
#
![](./img/Jenkins_Get_started.PNG)
#
![](./img/Jenkins_Customize.PNG)
#
![](./img/Jenkins_setup.PNG)
#
![](./img/Jenkins_pipeline.PNG)
#
![](./img/Jenkins_pipeline_config.PNG)
#
![](./img/Jenkins_pipeline_config1.PNG)
#
```
pipeline {
    agent any
    
    stages{
        stage("Code"){
            steps{
                echo "Clonning the code"
            }
        }
        stage("Build"){
            steps{
                echo "Building the code"
            }
        }
        stage("Push to Docker Hub"){
            steps{
                echo "Pushing the image to Docker Hub"
            }
        }
        stage("Deploy"){
            steps{
                echo "Deploying the container"
            }
        }
    }
}
```
#
![](./img/Jenkins_pipeline_build.PNG)
#
```
sudo usermod -aG docker jenkins
sudo reboot
```
#
![](./img/Docker_login.PNG)
#
![](./img/Credentials.PNG)
#
![](./img/Add_Credentials.PNG)
#
![](./img/Add_Credentials1.PNG)
#
![](./img/Add_Credentials2.PNG)
#
```
pipeline {
    agent any
    
    stages{
        stage("Code"){
            steps{
                echo "Clonning the code"
                git url:"https://github.com/Renatoksh/django-notes-app.git", branch: "main"
            }
        }
        stage("Build"){
            steps{
                echo "Building the code"
                sh "docker build -t notes-app ."
            }
        }
        stage("Push to Docker Hub"){
            steps{
                echo "Pushing the image to Docker Hub"
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                }
            }
        }
        stage("Deploy"){
            steps{
                echo "Deploying the container"
            }
        }
    }
}
```
#
### Build the job again to validate credentials work:
![](./img/Add_Credentials3.PNG)
#
```
pipeline {
    agent any
    
    stages{
        stage("Code"){
            steps{
                echo "Clonning the code"
                git url:"https://github.com/Renatoksh/django-notes-app.git", branch: "main"
            }
        }
        stage("Build"){
            steps{
                echo "Building the code"
                sh "docker build -t my-note-app ."
            }
        }
        stage("Push to Docker Hub"){
            steps{
                echo "Pushing the image to Docker Hub"
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker tag my-note-app ${env.dockerHubUser}/my-note-app:latest"
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker push ${env.dockerHubUser}/my-note-app:latest"
                }
            }
        }
        stage("Deploy"){
            steps{
                echo "Deploying the container"
            }
        }
    }
}
```
#
![](./img/Docker_push.PNG)
#
![](./img/DockerHub_push.PNG)
#
![](./img/Add_inbound_rule_8000.PNG)
#
```
pipeline {
    agent any
    
    stages{
        stage("Code"){
            steps{
                echo "Clonning the code"
                git url:"https://github.com/Renatoksh/django-notes-app.git", branch: "main"
            }
        }
        stage("Build"){
            steps{
                echo "Building the code"
                sh "docker build -t my-note-app ."
            }
        }
        stage("Push to Docker Hub"){
            steps{
                echo "Pushing the image to Docker Hub"
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker tag my-note-app ${env.dockerHubUser}/my-note-app:latest"
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker push ${env.dockerHubUser}/my-note-app:latest"
                }
            }
        }
        stage("Deploy"){
            steps{
                echo "Deploying the container"
                sh "docker run -d -p 8000:8000 renatoksh/my-note-app:latest"
            }
        }
    }
}
```
#
![](./img/Deploy.PNG)
#
![](./img/notes_working_first_time.PNG)
#
### Run again the build?
![](./img/Deploy_failed.PNG)
#
### To fix the issue, we need to install DockerCompose
#
```
sudo apt-get install docker-compose
```
#
### Cancel the installation of Docker-compose in case gets stuck, and try it again
![](./img/docker_compose_reinstalling.PNG)
#
### enter to the django-notes-app folder
```
cd django-notes-app
```
#
```
docker-compose up -d
```
### In case any error starting the docker-compose application, kill the process and try it again
![](./img/restarting_docker_compose.PNG)
#
![](./img/dokcer_ps.PNG)
#
### Resolution:
```
pipeline {
    agent any
    
    stages{
        stage("Code"){
            steps{
                echo "Clonning the code"
                git url:"https://github.com/Renatoksh/django-notes-app.git", branch: "main"
            }
        }
        stage("Build"){
            steps{
                echo "Building the code"
                sh "docker build -t my-note-app ."
            }
        }
        stage("Push to Docker Hub"){
            steps{
                echo "Pushing the image to Docker Hub"
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker tag my-note-app ${env.dockerHubUser}/my-note-app:latest"
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker push ${env.dockerHubUser}/my-note-app:latest"
                }
            }
        }
        stage("Deploy"){
            steps{
                echo "Deploying the container"
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}
```
#
### docker-compose.yml updated and commited
![](./img/docker-compose.yml_update.PNG)
#
### Confidure Declarative Checkout SCM in Jenkins job
![](./img/Configure_Declarative_checkout_SCM.PNG)
#
### Declarative Checkout SCM running and working
![](./img/Declarative_Checkout_SCM_working.PNG)
#
### Configure Webhook for Continuos Deployment
#
### Go to your Github repository django-notes-app
### Settings, Webhooks
![](./img/Webhook_configuration.PNG)
#
![](./img/Webhook_configuration1.PNG)

#
![](./img/Declarative_Checkout_SCM_Jenkins.PNG)

https://github.com/Renatoksh/1_handson_devops/assets/95065208/14ce5771-d820-420b-8ae2-eacc276fc2e4
