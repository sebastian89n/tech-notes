The **environment** in a Jenkins pipeline is a set of variables available to all pipeline steps. These variables can store configuration, credentials, or values used across stages. You can define them declaratively using the **`environment {}` block** or imperatively through the **`env` object** in a `script {}` block.

- **Global `environment {}`** (at pipeline level) → variables are defined once and available everywhere in the pipeline.
- **Stage-level `environment {}`** → variables exist only in that stage, overriding global ones temporarily.
- **`env.*` inside `script {}`** → allows setting or changing variables dynamically. Unlike stage-level `environment`, values set here persist globally across all later stages.

These variables are injected into the environment of all steps (e.g., `sh`, `bat`, `echo`), so you can reference them just like system environment variables.

```groovy
pipeline {
    agent any
    environment {
        GLOBAL_VAR = "global"
    }
    stages {
        stage('One') {
            environment {
                STAGE_VAR = "stage only"
            }
            steps {
                script {
                    env.DYNAMIC = "set in script"
                    echo "GLOBAL=$GLOBAL_VAR, STAGE=$STAGE_VAR, DYNAMIC=$DYNAMIC"
                }
            }
        }
        stage('Two') {
            steps {
                sh 'echo "GLOBAL=$GLOBAL_VAR, STAGE=$STAGE_VAR, DYNAMIC=$DYNAMIC"'
            }
        }
    }
}
```

**Result:**
- Stage One → has `GLOBAL_VAR`, `STAGE_VAR`, and `DYNAMIC`.
- Stage Two → `STAGE_VAR` disappears, but `GLOBAL_VAR` and `DYNAMIC` remain available.

In short: **`environment {}` is declarative and scoped, `env.*` is imperative and global**. Together they give you both stability (for fixed values) and flexibility (for runtime logic).