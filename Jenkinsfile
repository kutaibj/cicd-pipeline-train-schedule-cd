pipeline {
    agent any
	
	tools {
    jdk 'JDK 11'
  }
	
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true, // Select to mark the build as a failure if there is a problem publishing to a server. The default is to mark the build as unstable.
                        continueOnError: false, //Select to continue publishing to the other servers after a problem with a previous server.
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging', // Select an SSH configuration from the list configured in the global configuration of this Jenkins
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip', // Files to upload to a server.
                                        removePrefix: 'dist/', // First part of the file path that should not be created on the remote server.
                                        remoteDirectory: '/tmp', //This folder will be below the one in the global configuration, if present.
																//The folder will be created if does not exist
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }   
    }
}
