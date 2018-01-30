# Working with secrets

## Creating Secrets

Secrets are created using the Docker API.

```
# from a string
echo "Secret text" | docker secret create secret_name -

# from a file
docker secret create my_secret secret_file_name
```

Using the `prod` agent, you can create a Jenkinsfile in a repository containing
the secret files.

Example `Jenkinsfile`:
```
pipeline {
  agent {
    label "prod"
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '2'))
    disableConcurrentBuilds()
  }
  stages {
    stage("create") {
      when {
        branch "master"
      }
      steps {
        sh "docker secret create servicebus.rabbitmq.user servicebus.rabbitmq.user"
        sh "docker secret create servicebus.rabbitmq.password servicebus.rabbitmq.password"
      }
    }
  }
  post {
    always {
      sh "docker system prune -f"
    }
    failure {
      slackSend(
        color: "danger",
        message: "${env.JOB_NAME} failed: ${env.RUN_DISPLAY_URL}"
      )
    }
    success {
      slackSend(
        color: "good",
        message: "${env.JOB_NAME} succeeded: ${env.RUN_DISPLAY_URL}"
      )
    }
  }
}
```

Here is a full example using a fabricated username and password:
https://github.com/unbounded-tech/docker-secret-example


## npm package - get-docker-secrets

Github: https://github.com/patrickleet/get-docker-secrets

`get-docker-secrets` is a small library which reads docker secret file names,
and creates an object, with the filenames as keys, and their contents as value.

```
npm i get-docker-secrets --save
```