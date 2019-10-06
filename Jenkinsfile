def buildStatus = false
def testStatus = false

pipeline {
    agent any
    stages {
        stage ('Build & Test') {
            parallel {
                stage('Build') {
                    steps {
                        script {
                            try {
                                echo "Build Stage"
                                buildStatus = true
                            } catch (Exception err) {
                                buildStatus = false
                            }
                        }
                    }
                }
                stage('Test') {
                    steps {
                        script {
                            try {
                                echo "Test Stage"
                                testStatus = true
                            } catch(Exception err) {
                                testStatus = false
                            }
                        }
                    }
                }
            }
        }
        
        stage ('Packaging and Build Image') {
            when {
                expression {
                    buildStatus && testStatus
                }
            }
            parallel {
                stage('Build and push docker image') {
                    steps {
                        script {
                            echo "Dockerize Stage"
                        }
                    }
                }
                stage('Packaging Jar to tgz and save to alibaba') {
                    steps {
                        script {
                            echo "Packaging Stage"
                        }
                    }
                }
            }
        }   
        
        stage ('Deploy Feature/Alpha') {
            when {
                expression {
                    buildStatus && testStatus
                }
            }
            steps {
                script {
                    echo "This stage is underconstruction"
                    try {
                        echo "Run the command to Deploy feature/alpha"
                        try {
                            timeout(time: 1, unit: 'DAYS') {
                                env.userChoice = input message: 'Do you want to Release?',
                                parameters: [choice(name: 'Versioning Service', choices: 'no\nyes', description: 'Choose "yes" if you want to release this build')]
                            }
                            if (userChoice == 'no') {
                                echo "User refuse to release this build, stopping...."
                            }
                        } catch (Exception err) {
                            echo "Send notification to Slack that the operation has been aborted"
                        }
                    } catch (Exception err) {
                        echo "Will fail and send the report"
                    }
                }
            }
        }

        stage ('Release Build') {
            when {
                environment name: 'userChoice', value: 'yes'
            }
            steps {
                script {
                    try {
                        timeout(time: 1, unit: 'DAYS') {
                            versionName = input (
                                 id: 'version', message: 'Input version name', parameters: [
                                    [$class: 'TextParameterDefinition', description: 'Whatever you type here will be your version', name: 'Version']
                                ]
                            )
                        }
                      echo "the version will be: ${versionName}"
                    } catch (Exception err) {
                       echo "Send the notification to Slack that the operation has been canceled"
                    }
                }
            }
        }
    }
    environment {
        service_name = "${JOB_NAME}".split('/').first()
        // put yor environment in here
    }
}