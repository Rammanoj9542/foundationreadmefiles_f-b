# Jenkins and Docker Installation on Ubuntu

## Introduction

Jenkins is an open-source automation server that streamlines repetitive technical tasks in continuous integration and delivery of software. Docker, on the other hand, is a platform for developing, shipping, and running applications inside containers. This guide will walk you through installing Jenkins and Docker on Ubuntu 22.04, along with setting up a Jenkins pipeline for a foundation backend project.

### Prerequisites

To follow this tutorial, you'll need:
- One Ubuntu 22.04 server configured with a non-root sudo user and firewall.
- Oracle JDK 11 installed.

## Installing Jenkins

### Step 1: Add Repository Key

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo gpg --dearmor -o /usr/share/keyrings/jenkins.gpg

Step 2: Add Repository

sudo sh -c 'echo deb [signed-by=/usr/share/keyrings/jenkins.gpg] http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

Step 3: Update & Install Jenkins

sudo apt update

sudo apt install jenkins

Starting Jenkins

sudo systemctl start jenkins.service

sudo systemctl status jenkins

Opening the Firewall

sudo ufw allow 8080

sudo ufw status

###Setting Up Jenkins

Step 1: Jenkins Setup

Open Jenkins and navigate to "Manage Jenkins".

Click on "Global Tool Configuration" and add Docker.

Go to "Manage Plugins" and install necessary plugins for Docker integration.

Step 2: Create a New Job Setup

Open Jenkins and create a new job for the application.

Select "Pipeline" as the job type and click "OK".

Navigate to the Jenkins Dashboard and click on the created job to configure it.

Start writing the pipeline Groovy script.

After writing the script, click on "Apply" and then "Save".

Once saved, navigate back to the Jenkins Dashboard and click "Build Now".

You can view the progress of the build in the Stage View.

Go to the build history and click on the recent build number to view details.

Step 3: Clone Repository
Navigate to the Jenkins Dashboard.

Open the Jenkins pipeline job for your project.

Execute the "Clone Repository" stage to fetch the latest code from the repository.

###Installing Docker on Ubuntu

Step 1: Update & Install Dependencies

sudo apt update

sudo apt install docker.io

Step 2: Verify Installation
docker --version

Step 3: Pull & Run Hello-World Image
sudo docker run hello-world

###Jenkins Pipeline Script for Foundation Backend


pipeline {

    agent any
    
    stages {
    
        stage('Clone Local Directory') {
        
            steps {
            
                script {
                
                    def sourceDir = "/home/naveen/Desktop/Projects/Application_Backend_Foundation/"
                    
                    def destDir = "/home/naveen/.jenkins/workspace/Application_Backend_Foundation/"
                    
                    sh "rsync -av --exclude='__pycache__/' ${sourceDir} ${destDir}"
                    
                }
            }
        }
        stage('Check Dockerfile') {
        
            steps {
            
                script {
                
                    def dockerfilePath = "backendend/Dockerfile"
                    
                    if (fileExists(dockerfilePath)) {
                    
                        echo "Dockerfile found. Proceed with the process."
                        
                    } else {
                    
                        echo "Dockerfile not found. Stopping the process."
                        
                        error("Dockerfile not found.")
                        
                    }
                }
            }
        }
        
         stage('Build the Docker Image') {
         
            steps {
            
                script {
                
                    dir("frontend"){
                    
                        sh "docker build -t foundation/backend:1.1.1 ."
                        
                        echo "Frontend image built successfully"
                        
                    }
                }    
            }
        }
        
        stage('run the image'){
        
            steps{
            
                script{
                
                    sh "docker-compose up -d backendend"
                    
                    echo"frontend image run successfully"
                    
                }
            }
        }
        
        stage('docker imageslist'){
        
            steps{
            
                script{
                
                    sh 'docker images'
                    
                }
            }
        }
    }
}
---------------------------------------------------------------------------------------------------------------------------
##Jenkins Pipeline Script for Foundation Frontend

pipeline {

    agent any
    
    stages {
    
        stage('Clone Local Directory') {
        
            steps {
            
                script {
                
                    def sourceDir = "/home/naveen/Desktop/Projects/Application_Frontend_Foundation/"
                    
                    def destDir = "/home/naveen/.jenkins/workspace/Application_Frontend_Foundation/"
                    
                    sh "rsync -av --exclude='__pycache__/' ${sourceDir} ${destDir}"
                    
                }
            }
        }
        
        stage('Check Dockerfile') {
        
            steps {
            
                script {
                
                    def dockerfilePath = "frontend/Dockerfile"
                    
                    if (fileExists(dockerfilePath)) {
                    
                        echo "Dockerfile found. Proceed with the process."
                    } else {
                    
                        echo "Dockerfile not found. Stopping the process."
                        
                        error("Dockerfile not found.")
                        
                    }
                }
            }
        }
        
         stage('Build the Docker Image') {
         
            steps {
            
                script {
                
                    dir("frontend"){
                    
                        sh "docker build -t foundation/frontend:1.1.1 ."
                        
                        echo "Frontend image built successfully"
                        
                    }
                }    
            }
        }
        
        stage('run the image'){
        
            steps{
            
                script{
                
                    sh "docker-compose up -d frontend"
                    
                    echo"frontend image run successfully"
                    
                }
            }
        }
        
        stage('docker imageslist'){
        
            steps{
            
                script{
                
                    sh 'docker images'
                    
                }
            }
        }
        
        stage('curl test'){
        
            steps{
            
                script{
                
                    def serverUrl = 'http://172.25.1.223:3000'
                    
                    def curlOutput = sh(script: "curl -I ${serverUrl}", returnStdout: true).trim()
                    
                    if (curlOutput.contains('200 OK')) {
                    
                        echo "Server at ${serverUrl} is accessible."
                        
                    } else {
                    
                        error "Failed to access server ."
                        
                    }
                }    
            }
        }
    }
}
