# 📘 CI/CD Documentation – Jenkins with Prod Server & RDS Integration

This document describes the complete setup of a CI/CD pipeline using Jenkins, a production server, and AWS RDS (MariaDB) to deploy a full-stack application.

---

## 🚀 Step 1: Setup Jenkins Server

Provision a Jenkins server and install the required tools:
- Java
- Maven
- Jenkins

Ensure Jenkins is accessible via browser and initial setup is completed.

---

## 🖥️ Step 2: Setup Production Server (Prod Agent)

Create a separate server (EC2 or VM) to act as a **production server**.

On this server:
- Install Java
- Install Maven
- Configure it as a Jenkins agent with label:
  ```
  prod
  ```

Ensure Jenkins can successfully connect to this agent.

---

## 🔗 Step 3: Create Jenkins Pipeline

Create a new Jenkins pipeline and configure it to use the **prod agent**.

### 📌 Example Pipeline

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

## 🗄️ Step 4: Create MariaDB RDS Database

Create an AWS RDS instance using **MariaDB** engine.

- Configure database name as:
  ```
  student_db
  ```
- Ensure proper connectivity (security group, port access)

---

## 🔧 Step 5: Configure Backend with RDS

1. Copy the RDS endpoint from AWS.
2. Go to your backend GitHub repository.
3. Update `application.properties` with the following configuration:

```properties
server.port=8081

spring.datasource.url=jdbc:mariadb://database-1.czyeo2oc0o6y.ap-south-1.rds.amazonaws.com:3306/student_db?sslMode=trust
spring.datasource.username=admin
spring.datasource.password=admin123

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

4. Save and push the changes.

---

## 🌐 Step 6: Configure Frontend with Backend URL

1. Go to your frontend repository.
2. Update API base URL using:
   ```
   http://<PROD_PUBLIC_IP>:8081
   ```
3. Save and push the changes.

---

## 📦 Step 7: Install Node.js on Production Server

Install Node.js on the production server to support frontend build.

---

## ▶️ Step 8: Execute Pipeline

Run the Jenkins pipeline and monitor execution.

- Backend will be built and started
- Frontend will be built and deployed to web server

---

## 🔍 Step 9: Verify Application

- Access frontend via browser  
- Verify backend API is accessible on port **8081**

---

## ⚠️ Step 10: Manual Backend Start (If Needed)

If backend is not running after pipeline execution, start it manually on the production server:

```bash
cd backend/target
nohup java -jar student-registration-backend-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
```

---

## ✅ Outcome

- Jenkins pipeline configured with production agent  
- Backend deployed and connected to RDS  
- Frontend deployed and integrated with backend  
- Full application accessible via public IP  

---

## 🔒 Best Practices

- Use IAM roles instead of hardcoded credentials  
- Store configuration securely (avoid direct commits of secrets)  
- Monitor application logs regularly  
- Use system services or containers for production deployments  

---
