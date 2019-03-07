node {
    def app
    // Initialize a LinkedHashMap / object to share between stages
    def pipelineContext = [:]

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */
        checkout scm
    }

    //Install the npm dependecies locally for Nexus analysis
    stage('Install dependencies for OSCA') {
        sh 'npm install --production'
    }

    //Evaluate code with Nexus
    stage('OSCA') {
        //nexusPolicyEvaluation failBuildOnNetworkError: false, iqApplication: 'hellonode', iqScanPatterns: [[scanPattern: '**/*.js'],[scanPattern: '**/*.zip'],[scanPattern: '**/*.war'],[scanPattern: '**/*.ear'],[scanPattern: '**/*.tar.gz']], iqStage: 'build'
    }

    //Evaluate code with Cx only against certain vulnerability categories
    stage('SAST for High Risk') {
      	//step([$class: 'CxScanBuilder', comment: '', credentialsId: '', excludeFolders: 'node_modules', excludeOpenSourceFolders: '', exclusionsSetting: 'job', failBuildOnNewResults: false, failBuildOnNewSeverity: 'HIGH', filterPattern: '''!**/_cvs/**/*, !**/.svn/**/*,   !**/.hg/**/*,   !**/.git/**/*,  !**/.bzr/**/*, !**/bin/**/*,
      	//!**/obj/**/*,  !**/backup/**/*, !**/.idea/**/*, !**/*.DS_Store, !**/*.ipr,     !**/*.iws,
      	//!**/*.bak,     !**/*.tmp,       !**/*.aac,      !**/*.aif,      !**/*.iff,     !**/*.m3u, !**/*.mid, !**/*.mp3,
      	//!**/*.mpa,     !**/*.ra,        !**/*.wav,      !**/*.wma,      !**/*.3g2,     !**/*.3gp, !**/*.asf, !**/*.asx,
      	//!**/*.avi,     !**/*.flv,       !**/*.mov,      !**/*.mp4,      !**/*.mpg,     !**/*.rm,  !**/*.swf, !**/*.vob,
      	//!**/*.wmv,     !**/*.bmp,       !**/*.gif,      !**/*.jpg,      !**/*.png,     !**/*.psd, !**/*.tif, !**/*.swf,
      	//!**/*.jar,     !**/*.zip,       !**/*.rar,      !**/*.exe,      !**/*.dll,     !**/*.pdb, !**/*.7z,  !**/*.gz,
      	//!**/*.tar.gz,  !**/*.tar,       !**/*.gz,       !**/*.ahtm,     !**/*.ahtml,   !**/*.fhtml, !**/*.hdm,
      	//!**/*.hdml,    !**/*.hsql,      !**/*.ht,       !**/*.hta,      !**/*.htc,     !**/*.htd, !**/*.war, !**/*.ear,
      	//!**/*.htmls,   !**/*.ihtml,     !**/*.mht,      !**/*.mhtm,     !**/*.mhtml,   !**/*.ssi, !**/*.stm,
      	//!**/*.stml,    !**/*.ttml,      !**/*.txn,      !**/*.xhtm,     !**/*.xhtml,   !**/*.class, !**/*.iml, !Checkmarx/Reports/*.*''', fullScanCycle: 10, generatePdfReport: true, groupId: '22222222-2222-448d-b029-989c9070eb23', includeOpenSourceFolders: '', incremental: true, jobStatusOnError: 'UNSTABLE', osaArchiveIncludePatterns: '*.zip, *.war, *.ear, *.tgz', osaInstallBeforeScan: false, password: '{AQAAABAAAAAQz82giXfg/qmHdB6hYmJoUHmafrnOiSoy8DjtiI4LcwI=}', preset: '3', projectName: ///'hellonode', sastEnabled: true, serverUrl: '${CX_URL}', sourceEncoding: '1', thresholdSettings: 'global', username: '', vulnerabilityThresholdEnabled: true, vulnerabilityThresholdResult: 'FAILURE', waitForResultsEnabled: true])
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
         app = docker.build("${DOCKER_HUB_NAME}/hellonode")
	       pipelineContext.app = app
    }

    stage('Run image') {
      pipelineContext.dockerContainer = pipelineContext.app.run()
      //sh 'curl http://127.0.0.1:8000'
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }

    post {
      always {
        echo "Stop Docker image"
        script {
          if (pipelineContext && pipelineContext.dockerContainer) {
            pipelineContext.dockerContainer.stop()
			    }
		    }
	    }
    }
}

