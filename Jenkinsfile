// Initialize a LinkedHashMap / object to share between stages
def dockerContext = [:]
def zapContext = [:]

pipeline {

  //This job will run in any jenkins agent
  agent any

  environment {
    jobName = "${env.JOB_NAME}"
    dockerHub = "${DOCKER_HUB_NAME}"
    dockerImageTag = "${DOCKER_HUB_NAME}" + "/" + "${env.JOB_NAME}"
    zapContainerName = "ZAP"
    zapContainerID = ""
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



      //Instructs Docker to build the Dockerfile in the current directory w/ a tag
      stage('Build Project Container from Dockerfile') {
        steps {
          script {
            dockerImage = docker.build(dockerImageTag)
            dockerContext.dockerImage = dockerImage
            //sh 'docker build . -t ' + dockerImageTag
            echo 'SUCCESS: ' + jobName + ': Built Container: ' + dockerImageTag
          }
        }
      }

      stage('Run Containers') {
        parallel {

          //Instructs Docker to run the image interactively with a pseudo-tty, map the port 8000 in the container to port 8000 on my machine
          stage('Run Project Container') {
            steps {
              script {
                dockerContext.dockerContainer = dockerContext.dockerImage.run('-p 8000:8000')
                //sh 'docker run -p 8000:8000 ' + dockerImageTag
                projectIP = sh 'docker inspect -f \\\'\\{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}\\\' ' + dockerImageTag
                //projectIP = 'http://' + projectIP + ':8000'
                //echo 'SUCCESS: ' + jobName + ': Ran Container: ' + dockerImageTag + projectIP
              }
            }
          }

          //Build ZAP Docker container
          /*stage('Run ZAP Container') {
            steps {
              //Per https://github.com/stephendonner/docker-zap/blob/master/run-docker.sh
              sh 'docker run --name ' +zapContainerName+ ' -u zap -p 2375:2375 -d owasp/zap2docker-weekly zap.sh -daemon -port 2375 -host 127.0.0.1 -config api.disablekey=true -config scanner.attackOnStart=true -config view.mode=attack -config connection.dnsTtlSuccessfulQueries=-1 -config api.addrs.addr.name=.* -config api.addrs.addr.regex=true'
              echo 'SUCCESS: ' + jobName + ': Ran Container: ' + zapContainerName
            }
          }*/

        }
      }
/*
      stage('Perform ZAP Assessment') {
        steps {
          sh 'docker inspect -f ''{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'' ' + dockerImageTag
          sh 'docker exec ' +zapContainerName+ ' zap-cli -p 2375 status -t 120 && docker exec ' +zapContainerName+ ' zap-cli -p 2375 open-url $TARGET_URL'
          sh 'docker exec ' +zapContainerName+ ' zap-cli -p 2375 spider $TARGET_URL'
          sh 'docker exec ' +zapContainerName+ ' zap-cli -p 2375 active-scan -r $TARGET_URL'
          sh 'docker exec ' +zapContainerName+ ' zap-cli -p 2375 alerts'
          sh 'docker logs ' +zapContainerName

        }
      }
*/
    }

    post {
        always {
          script {
            if (dockerContext && dockerContext.dockerContainer) {
              dockerContext.dockerContainer.stop()
            }
            echo "Stop Docker image"
            //sh 'docker stop ' + dockerImageTag
            //sh 'docker rm ' + dockerImageTag
          }
        }
    }
}

