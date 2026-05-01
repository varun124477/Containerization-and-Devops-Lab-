```markdown
## Docker Lab – Experiment 7: CI/CD using Jenkins, GitHub and Docker Hub

### Aim
To design and implement a complete CI/CD pipeline using Jenkins, integrating source code from GitHub, and building & pushing Docker images to Docker Hub.

<hr>

<h4 align="center"> Pre‑requisite </h4>

- Docker & Docker Compose installed
- GitHub account (for source code)
- Docker Hub account (for image registry)
- Basic Git and Linux command knowledge

> **Project Directory Structure**  
> All source files, the Jenkins Compose file, and the experiment images are kept inside one folder.
>
> ![Local project files](./Images/image.png)

<hr>

### Step‑by‑Step Procedure

---

**Step 1 – Write the Jenkins Pipeline Definition (`Jenkinsfile`)**  
The `Jenkinsfile` contains the entire CI/CD pipeline written in Groovy.  
It defines four stages: Clone source, Build Docker image, Login to Docker Hub, and Push.

```groovy
pipeline {
    agent any
    environment {
        IMAGE_NAME = "varun99566/myapp"
    }
    stages {
        stage('Clone Source') {
            steps {
                git 'https://github.com/varun124477/my-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }
        stage('Login to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')]) {
                    sh 'echo $DOCKER_TOKEN | docker login -u varun99566 --password-stdin'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                sh 'docker push $IMAGE_NAME:latest'
            }
        }
    }
}
```
![Jenkinsfile content](./Images/1.png)

---

**Step 2 – Write the Flask Application (`app.py`)**  
A simple Python web application that returns a greeting message.

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from CI/CD Pipeline!"

app.run(host="0.0.0.0", port=80)
```
![app.py content](./Images/2.png)

---

**Step 3 – Create the Dockerfile**  
The Dockerfile packages the Flask app into a lightweight Python image.

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 80
CMD ["python", "app.py"]
```
![Dockerfile content](./Images/3.png)

---

**Step 4 – First Push Attempt to GitHub**  
After creating all files, the project is pushed to the remote repository.

```powershell
git push -u origin main --force
```
![Push command](./Images/4.png)

---

**Step 5 – Successful Push to GitHub**  
The force push uploads all source code and sets the upstream branch.

```powershell
git push -u origin main --force
```
![Successful push output](./Images/5.png)

---

**Step 6 – Verify the GitHub Repository**  
The repository now contains `app.py`, `Dockerfile`, `Jenkinsfile`, and `requirements.txt`.

![GitHub repository files](./Images/7.png)

---

**Step 7 – Write Docker Compose for Jenkins**  
Create `docker-compose.yml` to run Jenkins with persistent volume and Docker socket access.

```yaml
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: always
    ports:
      - "9000:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    user: root
volumes:
  jenkins_home:
```
![docker-compose.yml](./Images/6.png)

---

**Step 8 – Start Jenkins Container**  
Launch the Jenkins service using Docker Compose.

```powershell
docker compose up -d
```
![Starting Jenkins container](./Images/8.png)

---

**Step 9 – Jenkins Unlock Page**  
Navigate to `http://localhost:9000` (port 9000 as mapped). The unlock page asks for the initial admin password.

![Unlock Jenkins page](./Images/9.png)

---

**Step 10 – Retrieve Initial Admin Password**  
Fetch the password from inside the running Jenkins container.

```powershell
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
![Retrieving password](./Images/10.png)

---

**Step 11 – Unlock Jenkins**  
Paste the retrieved password and click **Continue**.

![Entering password](./Images/11.png)

---

**Step 12 – Install Suggested Plugins**  
Select **Install suggested plugins** to install Git, Pipeline, and other essential plugins.

![Plugin selection](./Images/12.png)

---

**Step 13 – Plugin Installation Progress**  
Wait while the plugins are downloaded and installed.

![Plugin installation list](./Images/13.png)

---

**Step 14 – Handle Plugin Failures**  
Some plugins may fail to install (non‑critical for this lab). Click **Continue** to proceed.

![Failed plugins](./Images/14.png)

---

**Step 15 – Create First Admin User (Blank Form)**  
Fill in a username, password, and full name for the administrator account.

![Admin user form](./Images/15.png)

---

**Step 16 – Admin User Details Entered**  
Complete the form and save.

![Filled admin user details](./Images/16.png)

---

**Step 17 – Configure Jenkins Instance URL**  
Accept the default URL (`http://localhost:9000/`) and finish the setup.

![Instance configuration](./Images/17.png)

---

**Step 18 – Jenkins Setup Complete**  
You are now ready to use Jenkins.

![Setup finished](./Images/18.png)

---

**Step 19 – Jenkins Dashboard**  
The main dashboard appears. You can now create jobs.

![Dashboard](./Images/19.png)

---

**Step 20 – Verify Installed Plugins (Optional)**  
Check that **Docker Pipeline** and **Git** plugins are present.

![Plugins verification](./Images/20.png)

---

**Step 21 – Create a New Pipeline Job**  
- Click **New Item**, enter `ci-cd-pipeline`, select **Pipeline**, and click OK.  
- Under **Pipeline**, choose **Pipeline script from SCM**.  
- Set **SCM** to **Git**.

![Pipeline initial configuration](./Images/21.png)

---

**Step 22 – Set GitHub Repository URL**  
Enter the repository URL (`https://github.com/varun124477/my-app.git`) and set **Script Path** to `Jenkinsfile`. Save the job.

![Repository URL configured](./Images/22.png)

---

**Step 23 – Trigger a Manual Build**  
Go to the pipeline job and click **Build Now**.

![Build Now button](./Images/23.png)

---

**Step 24 – Restart Jenkins Environment (Optional)**  
If needed, you can take down and re‑create the Jenkins container, or access its bash shell.

```powershell
docker compose down
docker compose up -d
docker exec -it jenkins bash
```
![Restarting Jenkins](./Images/24.png)

---

**Step 25 – Install Docker CLI Inside Jenkins (Optional)**  
Installing the Docker client inside the container is **not required** because the host Docker socket is mounted. However, it can be installed for debugging.

```bash
apt update && apt install -y docker.io
```
![Docker installation inside Jenkins](./Images/25.png)

---

**Step 26 – Successful Pipeline Execution**  
The console output shows all four stages: Clone → Build → Login → Push. The image is pushed to Docker Hub.

![Pipeline console success](./Images/26.png)

---

**Step 27 – Build History**  
The pipeline job page displays the build history, with the latest successful build highlighted.

![Build history](./Images/27.png)

---

**Step 28 – Verify Image on Docker Hub**  
The `varun99566/myapp` repository now contains the freshly pushed image.

![Docker Hub repository](./Images/28.png)

---

**Step 29 – Run the Docker Image from Docker Hub**  
Pull and run the image locally. The container exposes port 80, mapped to port 5000 on the host.

```powershell
docker run -d -p 5000:80 varun99566/myapp
```
![Running the container](./Images/29.png)

---

**Step 30 – Test the Application**  
Open a browser and navigate to `http://localhost:5000`. You should see the message **“Hello from CI/CD Pipeline!”**.

![Application output in browser](./Images/30.png)

<hr>

### 🔍 Key Concepts

- **Jenkinsfile** – Defines the pipeline as code; stored alongside the application.
- **Webhook (optional)** – Set up in GitHub to trigger Jenkins automatically on every push.
- **Credentials Store** – Docker Hub token is stored securely in Jenkins; `withCredentials` injects it temporarily.
- **Host Docker Socket** – Jenkins container shares the host’s Docker daemon via `-v /var/run/docker.sock`, allowing it to build and push images without running Docker inside.

---

### 📘 Notes

- The `version` warning from Docker Compose can be ignored (it is obsolete and can be removed).
- In the provided setup, Jenkins is accessed on **port 9000** instead of the default 8080.
- The pipeline uses `git push --force` only during initial setup; normally, regular pushes are sufficient.
```