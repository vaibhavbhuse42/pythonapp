# 🚀 Python App CI/CD Deployment using Jenkins & AWS EC2  

This project demonstrates a **fully automated CI/CD pipeline** for deploying a **Python web application** to an AWS EC2 instance using **Jenkins**.

Whenever you push new code to GitHub, Jenkins automatically:
1. Clones the repository  
2. Uploads the code to the EC2 server  
3. Installs dependencies  
4. Runs the app automatically 🎯  

---

## 🧩 Architecture Diagram  

Developer → GitHub → Jenkins → AWS EC2 → Python App → Browser

![](/image/Gemini_Generated_Image_iac2vqiac2vqiac2.png)

## ⚙️ Tools and Technologies
Component	Tool
Language	Python
Framework	Flask
CI/CD Tool	Jenkins
Cloud Platform	AWS EC2 (Ubuntu)
Version Control	Git & GitHub
SSH Agent	Jenkins Credentials Plugin
### 🖥️ Step 1: Setup AWS EC2 Instance

#### Launch an Ubuntu EC2 instance

#### Connect to it using SSH:

ssh -i your-key.pem ubuntu@<EC2-Public-IP>

#### Update and install Python:

sudo apt update -y
sudo apt install -y python3 python3-pip python3-venv git


#### Allow inbound port 5000 (Flask default port) in AWS Security Group.

## 🧱 Step 2: Create Python Application

#### app.py

from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "🚀 Hello from Flask App - CI/CD via Jenkins!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)


#### requirements.txt

flask


### 📸 Screenshot Example:

![](/image/Screenshot%20(14).png)

## 🧰 Step 3: Setup Jenkins

Install Jenkins (on another EC2 or your local system):

sudo apt update -y
sudo apt install openjdk-17-jre -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update -y
sudo apt install jenkins -y
sudo systemctl start jenkins


#### Access Jenkins at:

http://<jenkins-server-ip>:8080


#### Unlock Jenkins:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword


#### Install the following plugins:

Git Plugin

SSH Agent Plugin

Pipeline Plugin

## 🔐 Step 4: Configure SSH Credentials in Jenkins

Go to: Manage Jenkins → Credentials → Global → Add Credentials

Select type: SSH Username with private key

Username: ubuntu

Private key: paste content of your .pem key

ID: node-app-key

#### 📸 Example:

![](/image/Screenshot%20(9).png)

## 🧾 Step 5: Jenkinsfile

Below is the Jenkinsfile used in this project:

pipeline {
    agent any

    environment {
        SSH_CRED = 'node-app-key'
        SERVER_IP = '43.204.103.119'
        REMOTE_USER = 'ubuntu'
        APP_DIR = '/home/ubuntu/pythonapp'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/vaibhavbhuse42/pythonapp.git', branch: 'main'
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(credentials: ["${SSH_CRED}"]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} "mkdir -p ${APP_DIR}"
                        scp -r Dockerfile README.md app.py requirements.txt test ${REMOTE_USER}@${SERVER_IP}:${APP_DIR}/
                    '''
                }
            }
        }

        stage('Install & Run App') {
            steps {
                sshagent(["${SSH_CRED}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${SERVER_IP} '
                            sudo apt update &&
                            sudo apt install -y python3-venv python3-pip &&
                            cd ${APP_DIR} &&
                            python3 -m venv venv &&
                            source venv/bin/activate &&
                            pip install --upgrade pip &&
                            pip install -r requirements.txt &&
                            nohup python3 app.py --host=0.0.0.0 > app.log 2>&1 &
                            exit 0
                        '
                    """
                }
            }
        }
    }
    ``


#### 📸 Example:

![](/image/Screenshot%20(13).png)

## ⚡ Step 6: Create Jenkins Pipeline

Open Jenkins → New Item

Enter project name: python-app-deployment

Choose Pipeline

Under Pipeline Definition, select Pipeline script from SCM

SCM: Git

Repository URL:

https://github.com/vaibhavbhuse42/pythonapp.git


Branch: main

Save and click Build Now 🚀

### 📸 Example:

![Jenkins Build](/image/Screenshot%20(15).png)

## 🧠 Step 7: Validate Deployment

After successful build, open your browser:

http://<EC2-Public-IP>:5000


#### Expected Output:

🚀 Hello from Flask App - CI/CD via Jenkins!


#### 📸 Example:

![App Running](screenshots/live-app.png)

## 🔁 Pipeline Flow Summary
Stage	Description
Clone Repo	Fetches latest code from GitHub
Deploy to Server	Copies project files to EC2
Install & Run App	Installs dependencies and runs Flask app
Output	Live running Python app on EC2

#### 📸 Example:

![Pipeline Stages](/image/Screenshot%20(16).png)

## 🏁 Conclusion

This project implements end-to-end CI/CD automation for a Python Flask web app using:

Jenkins Pipeline

GitHub Integration

AWS EC2 SSH Deployment

Flask & Python Virtual Environment

Every new Git push triggers Jenkins, which automatically deploys the updated version to EC2 —
making deployment fully automated and production-ready 🚀

## 📂 Project Structure
pythonapp/
├── app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── README.md
└── test/

## 💬 Author

### 👨‍💻 Vaibhav Navnath Bhuse
💼 DevOps Engineer | 🧠 Cloud Enthusiast | 🚀 CI/CD Automation
📧 Email: your-email@example.com

🌐 GitHub: vaibhavbhuse42