# 📘 CI/CD Documentation – Jenkins, S3, Prod Server, RDS & SonarQube Integration

This document describes the complete CI/CD setup including Jenkins, S3 artifact storage, production deployment, AWS RDS integration, and SonarQube code quality analysis.

---

## 🚀 Step 1: Setup Jenkins Server
Provision Jenkins with Java and Maven and complete initial setup.

---

## 🖥️ Step 2: Setup Production Server (Prod Agent)
Install Java and Maven, connect as Jenkins agent with label `prod`.

---

## 🔗 Step 3: Create Jenkins Pipeline
Use prod agent for deployment.

```groovy
pipeline {
    agent {
        label 'prod'
    }
    stages {
        stage('Build') {
            steps {
                git branch: 'main', url: 'https://github.com/Milind2803/EasyCRUD-Updated-k8s.git'
            }
        }
        stage('Test') {
            steps {
                sh '''cd backend
                    mvn clean package -DskipTests
                    cd target
                    nohup java -jar student-registration-backend-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
                    '''
            }
        }
        stage('frontend') {
            steps {
                sh '''
                    cd frontend
                    npm install
                    npm run build
                    sudo cp -rf dist/* /var/www/html/
                    '''
            }
        }
    }
}
```

---

## ☁️ Step 4: Configure S3 Bucket
Create S3 bucket for artifact storage.

---

## 🔌 Step 5: Install Required Plugins for S3
Install:
- S3 Publisher  
- Artifact Manager on S3  
- Pipeline: AWS Steps  

---

## 🔑 Step 6: Configure AWS Credentials
Add AWS credentials in Jenkins (`aws-creds`).

---

## ⚙️ Step 7: Configure AWS Settings
Set region and credentials in Jenkins system settings.

---

## 📦 Step 8: Add S3 Upload Stage

```groovy
stage('Upload to S3') {
    steps {
        withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
            s3Upload(
                bucket: 'your-bucket-name',
                file: 'backend/target/student-registration-backend-0.0.1-SNAPSHOT.jar',
                path: 'artifacts/app.jar'
            )
        }
    }
}
```

---

## 🗄️ Step 9: Create MariaDB RDS
Database name: `student_db`

---

## 🔧 Step 10: Configure Backend

```properties
server.port=8081

spring.datasource.url=jdbc:mariadb://database-1.czyeo2oc0o6y.ap-south-1.rds.amazonaws.com:3306/student_db?sslMode=trust
spring.datasource.username=admin
spring.datasource.password=admin123

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

## 🌐 Step 11: Configure Frontend
Use backend URL:  
http://<PROD_PUBLIC_IP>:8081

---

## 📦 Step 12: Install Node.js
Install Node.js on prod server.

---

## ▶️ Step 13: Run Pipeline
Execute Jenkins pipeline.

---

## 🔍 Step 14: Verify Application
Check frontend and backend (port 8081).

---

## ⚠️ Step 15: Manual Backend Start
```bash
cd backend/target
nohup java -jar student-registration-backend-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
```

---

# 🔍 SonarQube Setup

## 16. Setup SonarQube EC2 Server
Install Docker and run SonarQube container.

## 17. Access Jenkins & SonarQube
- Jenkins: http://<jenkins-public-ip>:8080  
- SonarQube: http://<sonarqube-public-ip>:9000  

Login with admin/admin and change password.

---

## 18. Configure SonarQube
- Create Webhook  
- Create Project (studentapp)  
- Generate Token  

---

## 19. Configure Jenkins
- Install SonarQube Scanner plugin  
- Add Secret Text credential (sonar-token)  
- Configure Sonar server (Sonar-env)  

---

## 20. Create Jenkins Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('pull') {
            steps {
                git branch: 'main', url: 'https://github.com/Rohit-1920/EasyCRUD-Updated.git'
            }
        }
        stage('build') {
            steps {
                sh '''cd backend
                    mvn clean package -DskipTests'''
            }
        }
        stage('test') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'Sonar-env'){
                    sh '''cd backend
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                        -Dsonar.projectKey=studentapp \
                        -Dsonar.projectName=studentapp'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
    }
}
```

---

## 21. Run the Pipeline
Build job and verify results in SonarQube dashboard.

---

## ✅ Outcome
- Jenkins CI/CD  
- S3 artifact storage  
- RDS integration  
- Deployment  
- SonarQube analysis  
