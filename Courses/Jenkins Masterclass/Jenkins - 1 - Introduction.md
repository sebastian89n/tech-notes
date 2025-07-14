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

**When using docker in pipeline: **

![[Pasted image 20250714084851.png]]
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

### Workspace

In Jenkins, the **workspace** is shared per job on a given agent, not per build. This means it reflects the **latest run's state**, even when viewed from an older build. In contrast, **artifacts** are files archived at the end of a build and are **preserved per build**, so you can reliably access the exact output from that specific execution.

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
Note: it also has better performance 2s before vs 370ms after combining.

### Exit codes

```groovy
Display
+ echo Keyboard
+ cat build/computer.txt
Mainboard
Display
Keyboard
+ rm -rf build
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage[](http://localhost:8080/job/Laptop%20assembly/10/console#)
[Pipeline] { (Test)[](http://localhost:8080/job/Laptop%20assembly/10/console#)
[Pipeline] echo[](http://localhost:8080/job/Laptop%20assembly/10/console#)
Testing a new laptop
[Pipeline] sh[](http://localhost:8080/job/Laptop%20assembly/10/console#)
+ test -f build/computer.txt
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
ERROR: script returned exit code 1
```

**CLI command/tool - Jenkins codes:**
Exit code: 0 - success
Exit code: 1 .. 255 - error

### Variables

In Jenkins pipelines, you can define variables using the `def` keyword (e.g., `def myVar = 'value'`) or set environment variables using the `environment` block in Declarative pipelines. Variables defined with `def` are local to the script or stage, while environment variables (e.g., `env.MY_VAR`) are accessible across all steps and stages. You can also use built-in variables like `env.BUILD_NUMBER` or `env.JOB_NAME` to reference Jenkins context during the build.

In Groovy **single quotes `'...'` create literal strings**, so variables inside them are not evaluated. If you want to **interpolate variables**, you need to use **double quotes `"..."`**, which allow expressions like `"Build number is ${BUILD_NUMBER}"`. This is important when constructing dynamic paths or messages in Jenkins.

```groovy
pipeline {
    agent any

    environment {
        BUILD_FILE_NAME = 'laptop.txt'
    }

    stages {
        stage('Build') {
            steps {
                cleanWs()
                echo 'Building a new laptop ...'
                sh '''
                    echo $BUILD_FILE_NAME
                    mkdir -p build
                    echo "Mainboard" >> build/$BUILD_FILE_NAME
                    cat build/$BUILD_FILE_NAME
                    echo "Display" >> build/$BUILD_FILE_NAME
                    cat build/$BUILD_FILE_NAME
                    echo "Keyboard" >> build/$BUILD_FILE_NAME
                    cat build/$BUILD_FILE_NAME
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Testing the new laptop ...'
                sh '''
                    test -f build/$BUILD_FILE_NAME
                    grep "Mainboard" build/$BUILD_FILE_NAME
                    grep "Display" build/$BUILD_FILE_NAME
                    grep "Keyboard" build/$BUILD_FILE_NAME
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

**Declarative:**
```groovy
pipeline {
    agent any

    environment {
        MY_ENV_VAR = 'Hello'
    }

    stages {
        stage('Example') {
            steps {
                script {
                    def localVar = 'World'
                    echo "${env.MY_ENV_VAR}, ${localVar}!"
                }
            }
        }
    }
}
```

**Scripted:**
```groovy
node {
    def name = 'Solomon'
    env.GREETING = 'Hi'

    stage('Greet') {
        echo "${env.GREETING}, ${name}!"
    }
}

```
### Two main types of Jenkins pipelines

#### Declarative Pipeline (uses `pipeline`, `stages`, etc.)

- **Structure-first**, easier to read and maintain.    
- Recommended for most use cases.
- Declarative Pipeline Example (clean, structured):
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
    }
}

```

- Benefits:
	- Cleaner syntax.
	- Jenkins can validate structure before running.
	- Better UI integration and visualization.

#### **Scripted Pipeline** (uses `node`, `stage`, etc.)
- **Fully Groovy-based**, more flexible but verbose.
- You write everything as code blocks inside `node`.
- Scripted Pipeline Example (more flexible):
```groovy
node {
    stage('Build') {
        echo 'Building...'
    }

    stage('Test') {
        echo 'Testing...'
    }
}
```

Benefits:
- More power and control.
- Useful when you need complex logic or shared libraries.

|Feature|Declarative|Scripted|
|---|---|---|
|Syntax style|YAML-like, structured|Full Groovy|
|Starts with|`pipeline { ... }`|`node { ... }`|
|Simplicity|Easier to use/learn|More complex|
|Flexibility|Limited|Very flexible|
|Use case|Standard CI/CD|Custom logic-heavy flows|
#### What Scripted Can Do That Declarative Can’t (easily)

Let’s say you want to dynamically create stages based on files in a directory — **Declarative can’t do this directly**, but Scripted can:
```groovy
node {
    def testFiles = findFiles(glob: 'tests/*.txt')

    for (file in testFiles) {
        stage("Test ${file.name}") {
            echo "Running test with ${file.name}"
            sh "cat tests/${file.name}"
        }
    }
}
```