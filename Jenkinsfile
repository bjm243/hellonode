pipeline {

  //This job will run in any jenkins agent
  agent any

  environment {
    dockerContext = "" // Initialize a LinkedHashMap / object to share between stages
    jobName = "${env.JOB_NAME}"
    buildStatus = "FAILURE"
  }


    //Stages is the inicialization from pipeline steps.
    stages {

      /* Let's make sure we have the repository cloned to our workspace */
      stage('Clone Repository') {
        steps {
          checkout scm
          echo 'SUCCESS: ' + jobName + ': Cloned Repository'
        }
      }

      //Install the npm dependecies
      stage('Install Dependencies for OSCA') {
        steps {
          /*sh 'rmdir node_modules /s /q'*/
          sh 'npm install'
          echo 'SUCCESS: ' + jobName + ': Installed Dependencies for OSCA'
        }
      }

      //Evaluate code with Nexus
      //**Does not seem to evaluate entire workspace...
      stage('Perform OSCA') {
          steps {
            nexusPolicyEvaluation failBuildOnNetworkError: false, iqApplication: 'hellonode',
              iqScanPatterns: [[scanPattern: '**/*.js'],[scanPattern: '**/*.zip'],[scanPattern: '**/*.war'],[scanPattern: '**/*.ear'],[scanPattern: '**/*.tar.gz']],
              iqStage: 'build'
            echo 'SUCCESS: ' + jobName + ': Performed OSCA'
          }
      }

    }
}

