pipeline {

  //This job will run in any jenkins agent
  agent any

  environment {
    dockerContext = "" // Initialize a LinkedHashMap / object to share between stages
    jobName = "${env.JOB_NAME}"
    dockerHub = "${DOCKER_HUB_NAME}"
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

      stage('Perform AppSec Analysis') {
        parallel {
          //Evaluate code with Nexus
          stage('Perform OSCA') {
              steps {
                nexusPolicyEvaluation failBuildOnNetworkError: false, iqApplication: 'hellonode',
                  iqScanPatterns: [[scanPattern: '**/*.js'],[scanPattern: '**/*.zip'],[scanPattern: '**/*.war'],[scanPattern: '**/*.ear'],[scanPattern: '**/*.tar.gz']],
                  iqStage: 'build'
                echo 'SUCCESS: ' + jobName + ': Performed OSCA'
              }
          }

          //Evaluate code with Cx only against certain vulnerability categories
          stage('Perform SAST for High Risk') {
            steps {
              step([$class: 'CxScanBuilder', comment: '', credentialsId: '', excludeFolders: 'node_modules', excludeOpenSourceFolders: '', exclusionsSetting: 'job', failBuildOnNewResults: false, failBuildOnNewSeverity: 'HIGH', filterPattern: '''!**/_cvs/**/*, !**/.svn/**/*,   !**/.hg/**/*,   !**/.git/**/*,  !**/.bzr/**/*, !**/bin/**/*,
            	!**/obj/**/*,  !**/backup/**/*, !**/.idea/**/*, !**/*.DS_Store, !**/*.ipr,     !**/*.iws,
            	!**/*.bak,     !**/*.tmp,       !**/*.aac,      !**/*.aif,      !**/*.iff,     !**/*.m3u, !**/*.mid, !**/*.mp3,
            	!**/*.mpa,     !**/*.ra,        !**/*.wav,      !**/*.wma,      !**/*.3g2,     !**/*.3gp, !**/*.asf, !**/*.asx,
            	!**/*.avi,     !**/*.flv,       !**/*.mov,      !**/*.mp4,      !**/*.mpg,     !**/*.rm,  !**/*.swf, !**/*.vob,
            	!**/*.wmv,     !**/*.bmp,       !**/*.gif,      !**/*.jpg,      !**/*.png,     !**/*.psd, !**/*.tif, !**/*.swf,
            	!**/*.jar,     !**/*.zip,       !**/*.rar,      !**/*.exe,      !**/*.dll,     !**/*.pdb, !**/*.7z,  !**/*.gz,
            	!**/*.tar.gz,  !**/*.tar,       !**/*.gz,       !**/*.ahtm,     !**/*.ahtml,   !**/*.fhtml, !**/*.hdm,
            	!**/*.hdml,    !**/*.hsql,      !**/*.ht,       !**/*.hta,      !**/*.htc,     !**/*.htd, !**/*.war, !**/*.ear,
            	!**/*.htmls,   !**/*.ihtml,     !**/*.mht,      !**/*.mhtm,     !**/*.mhtml,   !**/*.ssi, !**/*.stm,
            	!**/*.stml,    !**/*.ttml,      !**/*.txn,      !**/*.xhtm,     !**/*.xhtml,   !**/*.class, !**/*.iml, !Checkmarx/Reports/*.*''', fullScanCycle: 10, generatePdfReport: false, groupId: '22222222-2222-448d-b029-989c9070eb23', includeOpenSourceFolders: '', incremental: true, jobStatusOnError: 'UNSTABLE', osaArchiveIncludePatterns: '*.zip, *.war, *.ear, *.tgz', osaInstallBeforeScan: false, password: '{AQAAABAAAAAQz82giXfg/qmHdB6hYmJoUHmafrnOiSoy8DjtiI4LcwI=}', preset: '3', projectName: 'hellonode', sastEnabled: true, serverUrl: '${CX_URL}', sourceEncoding: '1', thresholdSettings: 'global', username: '', vulnerabilityThresholdEnabled: true, vulnerabilityThresholdResult: 'FAILURE', waitForResultsEnabled: true])

              echo 'SUCCESS: ' + jobName + ': Performed SAST for High Risk'
            }
          }
        }
      }

      //Install the npm dependecies
      stage('Build Container') {
        agent { dockerfile true }
        steps {
          sh 'docker build . -t ' + dockerHub + '/' + jobName
          echo 'SUCCESS: ' + jobName + ': Built Container: ' + dockerHub + '/' + jobName
        }
      }

    }
}

