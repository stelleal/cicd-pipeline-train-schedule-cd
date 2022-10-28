pipeline {

    agent any

    stages {
        stage("build") {
            steps {
                echo "Building the aplication..."
                sh "./gradlew build --no-daemon"
                archiveArtifacts artifacts: dist/trainSchedule.zip
            }
        }

        stage("staging") {
            when {
                branch "master"
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USER",
                                    encryptedPassphrase: "$PWD"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourcesFiles: "dist/trainSchedule.zip",
                                        removePrefix: "dist/",
                                        remoteDirectory: "/tmp",
                                        execCommand: "sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule"
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }

        stage("production") {
            when {
                branch "master"
            }
            steps {
                input "Does the  staging env is working?"
                milestone(1)

                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USER",
                                    encryptedPassphrase: "$PWD"
                                ],
                                transfers: [
                                    sshTransfer(
                                        sourcesFiles: "dist/trainSchedule.zip",
                                        removePrefix: "dist/",
                                        remoteDirectory: "/tmp",
                                        execCommand: "sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule"
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
