How to store and use secrets in Jenkins:

1. Add credentials  
    Manage Jenkins → Credentials → (choose domain/scope: Global or Folder) → **Add Credentials**.  
    Pick a type:

- **Username with password** (for basic auth)
- **Secret text** (API tokens)
- **SSH Username with private key**
- **Secret file** or **Certificate** if needed  
    Give it a clear **ID** (you’ll reference this in Jenkinsfiles).

2. Use them safely in a Pipeline (masked in logs)

**Username/Password**

```groovy
pipeline {
  agent any
  stages {
    stage('Use creds') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'my-login',
                                          usernameVariable: 'USER',
                                          passwordVariable: 'PASS')]) {
          sh 'curl -u "$USER:$PASS" https://example.com'
        }
      }
    }
  }
}
```

**Secret Text (e.g., API token)**

```groovy
withCredentials([string(credentialsId: 'netlify-token', variable: 'NETLIFY_AUTH_TOKEN')]) {
  sh 'npx netlify deploy --auth="$NETLIFY_AUTH_TOKEN"'
}
```

**Declarative shorthand (creates `*_USR`/`*_PSW`)**

```groovy
environment {
  MY_LOGIN = credentials('my-login')   // exposes MY_LOGIN_USR and MY_LOGIN_PSW
}
```

**Via environment variables:**
```groovy
environment {
	NETLIFY_SITE_ID = 'xyz'
	NETLIFY_AUTH_TOKEN = credentials('netlify-token')
}
```

**Tips:**
- Don’t `echo` secrets; Jenkins masks, but avoid accidental leaks.
- Scope credentials to the smallest needed level (Folder/Multibranch).
- Rotate tokens regularly and keep IDs stable so you don’t touch Jenkinsfiles.
- For Docker/agents, credentials are injected into the `sh` environment inside the step.