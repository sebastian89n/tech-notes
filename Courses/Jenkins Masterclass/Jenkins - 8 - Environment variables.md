In a pipeline job definition, environment variables can be configured to store values that your job’s steps can reuse without hardcoding them. In Jenkins Declarative Pipeline, you can define them globally under the `environment` block so they are available to all stages, or you can define them inside a specific stage so they only apply there.

**Global example:**

```groovy
pipeline {
    agent any
    environment {
        APP_ENV = 'production'
        DB_URL = credentials('db-connection')
    }
    stages {
        stage('Build') {
            steps {
                sh 'echo Environment: $APP_ENV'
            }
        }
    }
}
```

**Stage-scoped example:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            environment {
                BUILD_MODE = 'debug'
            }
            steps {
                sh 'echo Build mode: $BUILD_MODE'
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo No BUILD_MODE here unless redefined'
            }
        }
    }
}
```

Variables defined in the `environment` block are available to all stages, while stage-level `environment` blocks only affect that stage. In Scripted Pipeline, you can set them using `env.VARIABLE_NAME = 'value'`. For sensitive values, use Jenkins Credentials and reference them in the `environment` block instead of hardcoding.