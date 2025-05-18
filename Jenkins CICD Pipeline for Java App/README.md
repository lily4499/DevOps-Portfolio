
#  Automated CI/CD Pipeline for Java Web App using Jenkins

## 📘 Scenario

You are a DevOps Engineer at a fintech company. The dev team pushes code to a GitHub repo for a Java-based loan calculator web app. Your task is to automate the CI/CD pipeline to:

* Build the app with Maven
* Run unit tests
* Package it as a WAR file
* Deploy it to Tomcat running on an AWS EC2 instance
* Notify developers via Slack

---

## 🛠️ Tools Used

* **Jenkins** – CI/CD automation
* **GitHub** – Source code repository
* **Maven** – Java build tool
* **Tomcat** – Application server
* **AWS EC2** – Deployment environment
* **Slack/Email** – Notifications

---
## create_project.py

```
import os

# Define base directory
base_dir = '/home/lilia/VIDEOS/loan-calculator-java'

# Define file structure and contents
file_structure = {
    "src/main/java/com/example/LoanCalculator/LoanController.java": """
package com.example.LoanCalculator;

public class LoanController {
    public String calculateLoan(double amount, double rate, int years) {
        double interest = amount * rate * years / 100;
        return "Total repayment: $" + (amount + interest);
    }

    public static void main(String[] args) {
        LoanController controller = new LoanController();
        System.out.println(controller.calculateLoan(10000, 5, 2));
    }
}
""",
    "test/java/com/example/LoanCalculator/LoanControllerTest.java": """
package com.example.LoanCalculator;

import org.junit.Test;
import static org.junit.Assert.*;

public class LoanControllerTest {
    @Test
    public void testCalculateLoan() {
        LoanController controller = new LoanController();
        String result = controller.calculateLoan(10000, 5, 2);
        assertEquals("Total repayment: $11000.0", result);
    }
}
""",
    "pom.xml": """
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>loan-calculator</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>loan-calculator</finalName>
    </build>
</project>
""",
    "Jenkinsfile": """
pipeline {
    agent any

    environment {
        EC2_IP = "YOUR_EC2_PUBLIC_IP"
        APP_NAME = "loanapp"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/youruser/loan-calculator-java.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy to Tomcat on EC2') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'tomcat-ec2',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: '**/target/*.war',
                                    remoteDirectory: '/opt/tomcat/webapps/',
                                    removePrefix: 'target',
                                    execCommand: '/opt/tomcat/bin/shutdown.sh; sleep 5; /opt/tomcat/bin/startup.sh'
                                )
                            ],
                            usePromotionTimestamp: false,
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Notify via Slack') {
            steps {
                slackSend(channel: '#ci-cd', message: "Build & deployment of ${APP_NAME} completed ✅", color: '#36a64f')
            }
        }
    }

    post {
        failure {
            slackSend(channel: '#ci-cd', message: "Build failed for ${APP_NAME} ❌", color: '#FF0000')
        }
    }
}
"""
}

# Create directories and files
for relative_path, content in file_structure.items():
    file_path = os.path.join(base_dir, relative_path)
    os.makedirs(os.path.dirname(file_path), exist_ok=True)
    with open(file_path, 'w') as f:
        f.write(content.strip())

"Files created successfully at /home/lilia/VIDEOS/loan-calculator-java"

```

---
Save the script as create_project.py
Run it with sudo:
```bash
sudo python3 create_project.py

```
---



---

## 📁 Project Structure

```bash
loan-calculator-java/
├── src/
│   └── main/java/com/example/LoanCalculator/
│       └── LoanController.java
├── test/
│   └── java/com/example/LoanCalculator/
│       └── LoanControllerTest.java
├── pom.xml
├── Jenkinsfile
└── README.md
```

---

## ✅ Step-by-Step Implementation

###  Step 1: Launch AWS EC2 & Install Tomcat

```bash
# Launch EC2 instance
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --key-name my-key \
  --security-groups my-sg

# Connect to EC2
ssh -i my-key.pem ec2-user@<ec2-public-ip>

# Install Tomcat
sudo yum install java -y
wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.85/bin/apache-tomcat-9.0.85.tar.gz
tar -xvf apache-tomcat-9.0.85.tar.gz
sudo mv apache-tomcat-9.0.85 /opt/tomcat
chmod +x /opt/tomcat/bin/*.sh
/opt/tomcat/bin/startup.sh
```

✅ **Ensure port 8080 is open in the EC2 security group.**

---

###  Step 2: Set Up Jenkins

```bash
# Run Jenkins in Docker
docker run -d \
  -p 9090:8080 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins \
  jenkins/jenkins:lts
```

> Or install Jenkins directly on a server or VM.

---

###  Step 3: Install Jenkins Plugins

Install the following plugins via **Manage Jenkins → Plugin Manager**:

* Git Plugin
* Maven Integration Plugin
* Publish Over SSH
* Slack Notification Plugin

---

###  Step 4: Configure Jenkins

* **Maven:** `Manage Jenkins → Global Tool Configuration → Maven`
* **SSH:**
  Generate key and copy it to EC2:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa_jenkins
ssh-copy-id -i ~/.ssh/id_rsa_jenkins.pub ec2-user@<ec2-public-ip>
```

* In **Publish over SSH Plugin**, configure:

  * **Name**: `tomcat-ec2`
  * **Hostname**: `EC2 IP`
  * **Username**: `ec2-user`
  * **Key**: `~/.ssh/id_rsa_jenkins`
  * **Remote Dir**: `/opt/tomcat/webapps`

---

###  Step 5: Add Jenkinsfile to GitHub

```groovy
pipeline {
    agent any

    environment {
        EC2_IP = "YOUR_EC2_PUBLIC_IP"
        APP_NAME = "loanapp"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/youruser/loan-calculator-java.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy to Tomcat on EC2') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'tomcat-ec2',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: '**/target/*.war',
                                    remoteDirectory: '/opt/tomcat/webapps/',
                                    removePrefix: 'target',
                                    execCommand: '/opt/tomcat/bin/shutdown.sh; sleep 5; /opt/tomcat/bin/startup.sh'
                                )
                            ],
                            usePromotionTimestamp: false,
                            verbose: true
                        )
                    ]
                )
            }
        }

        stage('Notify via Slack') {
            steps {
                slackSend(channel: '#ci-cd', message: "Build & deployment of ${APP_NAME} completed ✅", color: '#36a64f')
            }
        }
    }

    post {
        failure {
            slackSend(channel: '#ci-cd', message: "Build failed for ${APP_NAME} ❌", color: '#FF0000')
        }
    }
}
```

---

###  Step 6: Create Jenkins Job

* Type: **Pipeline**
* Pipeline Script From SCM:

  * SCM: Git
  * Repo: `https://github.com/youruser/loan-calculator-java.git`
  * Script path: `Jenkinsfile`

---

###  Step 7: Trigger the Pipeline

Push code to GitHub → Jenkins will:

* Checkout code
* Build WAR with Maven
* Run unit tests
* Deploy WAR to EC2’s Tomcat
* Notify team via Slack

---

## 📤 Slack Notification Setup

* Install **Slack Notification Plugin**
* In **Manage Jenkins → Configure System → Slack**, add:

  * Workspace
  * Token Credential
  * Channel

Then call:

```groovy
slackSend(channel: '#ci-cd', message: "Deployment successful ✅")
```

---

##  Summary Table

| Component       | Purpose                          |
| --------------- | -------------------------------- |
| **GitHub**      | Hosts source code & Jenkinsfile  |
| **Jenkins**     | Automates CI/CD process          |
| **Maven**       | Builds and packages the Java app |
| **Tomcat**      | Hosts the deployed WAR app       |
| **EC2**         | Deployment target                |
| **Slack/Email** | Notification mechanism           |

---



