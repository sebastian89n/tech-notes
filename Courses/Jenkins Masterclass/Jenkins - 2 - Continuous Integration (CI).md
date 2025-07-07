>In DevOps, **Continuous Integration (CI)** is a key practice that automates the building and testing of code every time a change is committed to the repository. Tools like **Jenkins** enable CI by automatically pulling code, compiling it, running tests, and providing feedback to developers. This helps detect issues early, shortens development cycles, and supports faster, more reliable software delivery in a DevOps pipeline.

**Jenkins build environment**
- Install all tools needed (like Node.js / npm) on the Jenkins agent.
- Use docker with container

11ec5e7d6d551279ce3bf6971c8934fd00

```groovy
pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                sh 'echo "Without docker"'
            }
        }
        
        stage('w/ docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh 'echo "With docker"'
                sh 'npm --version'
            }
        }
    }
}
```

With docker, Jenkins will find an agent able to run docker images, it will pull the images and run commands **inside** docker image.

By default 2 stages will be run in a different workspaces.
To run it in a single workspace we need to configure `reuseNode true`

```groovy
pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                sh '''
                    echo "Without docker"
                    ls -la
                    touch cointainer-no.txt
                '''
            }
        }
        
        stage('w/ docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "With docker"
                    ls -la
                    touch container-yes.txt
                '''
            }
        }
    }
}
```

