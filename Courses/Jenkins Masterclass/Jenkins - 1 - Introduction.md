### Setup

```
docker run --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

### Architecture

**Jenkins Server (Controller)**
- manages execution of the pipelines and the associated jobs
- stores the result

**Jenkins Agent**
- Executes the job
- Server delegates action to agents

**Freestyle job:**
Freestyle jobs in Jenkins are configured through the web UI and are ideal for simple, linear build tasks. They offer limited flexibility and are harder to maintain for complex workflows. Each step is added manually, and reusability across projects is minimal.

**Pipeline Job:**
Pipeline jobs are defined in code using a `Jenkinsfile`, allowing for complex, multi-stage workflows and version-controlled automation. They support features like parallel execution, conditional logic, and shared libraries. Pipelines are better suited for modern DevOps and continuous delivery practices.
### First pipeline

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building a new laptop ...'
                sh 'mkdir -p build'
                sh 'touch build/computer.txt'
                sh 'echo "Mainboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Display" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Keyboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```

In Jenkins, a **workspace** is a directory on the build agent where job files are checked out and built. Each job gets its own workspace to avoid conflicts. It’s good practice to **clean the workspace** before or after builds—especially for long-lived jobs—to prevent issues from leftover files or outdated artifacts.

### Artifacts
**Artifacts** are files generated during a build process, such as compiled binaries, test results, or logs, that are archived for later use. They allow you to store and access outputs from builds, making it easier to analyze results, distribute software, or promote builds between environments. You can configure Jenkins to automatically archive specific files as artifacts at the end of a job

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                cleanWs()
                echo 'Building a new laptop ...'
                sh 'mkdir -p build'
                sh 'touch build/computer.txt'
                sh 'echo "Mainboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Display" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Keyboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
            }
        }
    }

    post {
        success{
            archiveArtifacts artifacts: 'build/**'
        }
    }
}
```

### Combining multiple shell scripts into one

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                cleanWs()
                echo 'Building a new laptop ...'
                sh '''
                    mkdir -p build
                    touch build/computer.txt
                    echo "Mainboard" >> build/computer.txt
                    cat build/computer.txt
                    echo "Display" >> build/computer.txt
                    cat build/computer.txt
                    echo "Keyboard" >> build/computer.txt
                    cat build/computer.txt
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'build/**'
        }
    }
}
```