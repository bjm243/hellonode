node {
    def app
    // Initialize a LinkedHashMap / object to share between stages
    def dockerContext = [:]
    def jobName = "${env.JOB_NAME}"
    def buildStatus

    cloneRepo()
}

/* Let's make sure we have the repository cloned to our workspace */
def cloneRepo() {
  stage 'Clone repository'
  checkout scm
  buildStatus = "SUCCESS: " + jobName + ": Repo cloned to workspace completed"
  echo buildStatus
  buildStatus = ""
}

def sendEmailNotification(status) {
  //office365ConnectorSend message: "<Your message>", status:'${status}', webhookUrl:"${O365_WEBHOOK}"
}

