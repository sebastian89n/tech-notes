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