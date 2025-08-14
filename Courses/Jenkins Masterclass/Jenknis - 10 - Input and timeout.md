In a Jenkins Pipeline, the `input` step is used to pause the execution and wait for manual approval or input from a user before continuing. This is often applied in deployment pipelines, for example, to require confirmation before promoting a build to production. The step can display a message, request parameters (like a choice or text), and only proceeds when someone with the right permissions provides the input. If no one approves within the timeout period, the pipeline can be configured to fail or abort.

The `timeout` step is used to limit how long a block of code or stage can run. If the specified time is exceeded, Jenkins will automatically abort the execution of that block and mark the step as failed (or continue, depending on configuration). This is useful for preventing jobs from hanging indefinitely due to stuck builds, waiting for unavailable resources, or unapproved `input` steps. It ensures pipelines stay predictable and resources are freed when something takes too long.

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building ...'
            }
        }
        stage('Approval') {
            steps {
	            // input 'Ready to deploy?'
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Ready to deploy?', ok: 'Yes, I am sure I want to deploy'
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}

```

![[Pasted image 20250814083901.png]]

On top of that logs will show who approved the step:
![[Pasted image 20250814084411.png]]

For playing with such options check pipeline syntax:
![[Pasted image 20250814084302.png]]