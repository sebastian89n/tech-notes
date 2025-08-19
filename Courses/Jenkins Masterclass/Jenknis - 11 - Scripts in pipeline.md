In a **declarative pipeline**, most tasks are expressed using high-level steps (e.g., `sh`, `checkout`, `stage`). However, sometimes you need more flexibility—such as using loops, conditionals, or custom Groovy logic. For these cases, you can use the **`script` block**, which lets you switch into the **imperative pipeline syntax**.

The `script` block acts as an escape hatch: it allows you to mix declarative pipeline structure with imperative Groovy code.

### Basic Example
```groovy
pipeline {
    agent any

    stages {
        stage('Example') {
            steps {
                script {
                    def name = "Sebastian"
                    echo "Hello, ${name}!"
                }
            }
        }
    }
}
```

Here, the variable assignment and string interpolation are done inside the `script` block, since they’re not possible directly in declarative steps.

Using Logic Inside `script`
```groovy
pipeline {
    agent any

    stages {
        stage('Build Matrix') {
            steps {
                script {
                    def versions = ["1.0", "2.0", "3.0"]
                    for (v in versions) {
                        echo "Building version ${v}"
                        // call other steps or custom functions here
                    }
                }
            }
        }
    }
}
```

### Best Practices
- Use `script` blocks **only when necessary**—prefer declarative steps when possible.
- Keep `script` blocks small and focused to maintain readability.
- Extract repeated logic into **shared libraries** instead of large `script` sections

---
### Declarative `script` vs Scripted Pipeline

- **scripted pipeline** is **entirely imperative**, written in Groovy from start to finish.
```groovy
node {
    stage('Build') {
        sh './gradlew build'
    }
    stage('Test') {
        sh './gradlew test'
    }
}
```

- A **declarative pipeline** is **structured**, with a fixed syntax (`pipeline {}`, `stages {}`, `steps {}`). Inside it, you can drop into a **`script {}` block** to run imperative Groovy when needed.
### Can `script` Replace Scripted Pipelines?
- **For small, localized logic** (loops, conditionals, helper functions), you can use `script` inside a declarative pipeline instead of switching to a full scripted pipeline. This is often enough for most modern CI/CD use cases.
    
- **For pipelines that are mostly dynamic or heavily rely on Groovy features**, wrapping everything in `script {}` inside declarative essentially turns it back into a scripted pipeline—with extra nesting. At that point, it’s cleaner and more maintainable to just write a **pure scripted pipeline**.
## Rule of Thumb
- Use **declarative pipelines** for readability, standardization, and maintainability.
- Use **`script` blocks inside declarative** for occasional flexibility.
- Use **scripted pipelines** only when your pipeline is inherently complex, dynamic, and can’t be expressed cleanly in declarative form.