2.  Simple Static Web page. 
TASK : CI and CD  
• Write the code in Visual Studio and push it to your GITHUB repository. 
• Modify the code and commit the updates back in GITHUB. 
• Using Jenkins configure the job as pipeline and create the stages for the CI and CD process. 
• Containerize the application using Dockerfile and display the stages in stage view in Jenkins. 
• Create kubernetes cluster and visualize the deployment in the given node port. 

a. The student results 
Open vscode -> folder student-results
Create index.html
<!DOCTYPE html>
<html>
<head>
    <title>Student Results</title>
    <style>
        body {
            font-family: Arial;
            text-align: center;
            background-color: #f2f2f2;
        }
        table {
            margin: auto;
            border-collapse: collapse;
            width: 60%;
        }
        th, td {
            border: 1px solid black;
            padding: 10px;
        }
        th {
            background-color: #4CAF50;
            color: white;
        }
    </style>
</head>
<body>
<h1>Student Results</h1>
<table>
    <tr>
        <th>Name</th>
        <th>Subject</th>
        <th>Marks</th>
        <th>Grade</th>
    </tr>
    <tr>
        <td>Srilaya</td>
        <td>Maths</td>
        <td>90</td>
        <td>A</td>
    </tr>
    <tr>
        <td>Rahul</td>
        <td>Science</td>
        <td>85</td>
        <td>A</td>
    </tr>
    <tr>
        <td>Anu</td>
        <td>English</td>
        <td>78</td>
        <td>B</td>
    </tr>
</table>
</body>
</html>
Open in browser and check locally
 
Create repo and push to it
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/Srilaya18/student.git
git push -u origin main
 

Create Dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80

Create deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: student-results
spec:
  replicas: 2
  selector:
    matchLabels:
      app: student-results
  template:
    metadata:
      labels:
        app: student-results
    spec:
      containers:
      - name: student-results
        image: srilayam/student-results:latest   # ⚠️ MUST match DockerHub image
        ports:
        - containerPort: 80
---          # ⚠️ IMPORTANT: YOU MISSED THIS LINE (VERY IMPORTANT)
apiVersion: v1
kind: Service
metadata:
  name: student-service
spec:
  type: NodePort
  selector:
    app: student-results
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007

Push to git again
git add .
git commit -m "Added Dockerfile and Kubernetes config"
git push
 

Install Kubernetes CLI plugin, Docker and Docker Pipeline 
Steps 
Manage Jenkins - Credentials - (System) -Global credentials (unrestricted) - Add Credentials 
Fill like this: 
Kind: Username with password 
Scope: Global 
Username: srilayam
Password: sridocker@123
ID: dockerhub-creds 
Description: DockerHub Login

Create pipeline and add script

pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "srilayam/student-results:latest"  //add docker username
    } 
    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Srilaya18/student.git'   //repo link change
            }
        }
        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }
        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    bat 'docker login -u %USER% -p %PASS%'
                    bat 'docker push %DOCKER_IMAGE%'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
            }
        }
    }
    post {
        success { 
            echo 'CI/CD Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check logs.'
        }
    }
}


 
   
In terminal
Check pods- kubectl get pods        (must get 1/1 then only working)
Check service- kubectl get svc 
 
Website link is- http://localhost:30007
