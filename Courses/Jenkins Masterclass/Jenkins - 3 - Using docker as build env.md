![[Pasted image 20250714084851.png]]

```groovy
pipeline {
    agent any

    stages {
		// This is a comment
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'Test stage'
                    test -e build/index.html
                    npm test
                '''
            }   
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
```

The `post { always { junit 'test-results/junit.xml' } }` block in a Jenkins Pipeline is a **standard way to publish test results** after the build completes, regardless of the outcome. The `junit` step processes test result files in **JUnit XML format**, which is a **widely adopted standard** across many testing frameworks (like JUnit, TestNG, PyTest, etc.). This standardization allows Jenkins to **consistently parse and visualize test results**, enabling features like test trend graphs, failure tracking, and integration with plugins. Because of its ubiquity, the JUnit format has become the de facto choice for test reporting in Jenkins CI pipelines.