pipeline {

  //This job will run in any jenkins agent
  agent any

  environment {
    dockerContext = [:] // Initialize a LinkedHashMap / object to share between stages
    jobName = "${env.JOB_NAME}"
    buildStatus = "FAILURE"
  }

    /* Let's make sure we have the repository cloned to our workspace */
    stages {
      stage('Clone Repository') {
        steps {
          checkout scm
          buildStatus = "SUCCESS: " + jobName + ": Repo cloned to workspace completed"
          echo buildStatus
          buildStatus = ""
        }
      }
    }

}

