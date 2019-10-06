def build_status = false
def test_status = false
def deploy_status = false
def release_status = false

pipeline {
    agent any

    environment {
        // service_name = "${JOB_NAME}".split('/').first()
        // build_tool = sh (
        //     script: ''' if [[ -f pom.xml ]]; then
        //                     echo 'mvnw'
        //                 elif [[ -f build.gradle ]]; then
        //                     echo 'gradlew'
        //                 fi ''',
        //     returnStdout: true
        // ).trim()
        // env_name = sh (
        //     script: ''' if [[ ${GIT_BRANCH} == *'feature'* ]] || [[ ${GIT_BRANCH} == *'hotfix'* ]] || [[ ${GIT_BRANCH} == *'bugfix'* ]]; then
        //                     echo 'feature'
        //                 elif [[ ${GIT_BRANCH} == *'master' ]]; then
        //                     echo 'alpha'
        //                 else
        //                     echo 'build not allowed in this branch'
        //                 fi ''',
        //     returnStdout: true
        // ).trim()
    }
    stages {
        stage ('Build & Test') {
            parallel {
                stage('Build') {
                    steps {
                        script {
                            try {
                                echo "Build Stage"
                                build_status = true
                            } catch (Exception err) {
                                build_status = false
                            }
                        }
                    }
                }
                stage('Test') {
                    steps {
                        script {
                            if (env.build_tool == "mvnw") {
                                try {
                                    echo "use mvnw"
                                    test_status = true
                                } catch (Exception err) {
                                    test_status = false
                                }
                            } else if (env.build_tool == "gradlew") {
                                try {
                                    echo "use gradlew"
                                    test_status = true
                                } catch (Exception err) {
                                    test_status = false
                                }
                            } else {
                                echo "Can't specify build tool, exitting...."
                                test_status = false
                            }
                        }
                    }
                }
            }
        }

        stage ('Packaging and Dockerize Image') {
            when {
                expression {
                    build_status && test_status
                }
            }
            parallel {
                stage('Build and push docker image') {
                    steps {
                        script {
                            echo "This stage will Dockerize the services"
                        }
                    }
                }
                stage('Packaging Jar to and store to alibaba OSS') {
                    steps {
                        script {
                            echo "This stage will package the services and store to alibaba OSS"
                        }
                    }
                }
            }
        }   
        
        stage ('Deploy') {
            when {
                expression {
                    build_status && test_status
                }
            }
            steps {
                script {
                    try {
                        echo "Run the command to Deploy"
                        deploy_status = true

                        if (env.env_name == 'alpha') {
                            try {
                                timeout(time: 1, unit: 'DAYS') {
                                    env.userChoice = input message: 'Do you want to Release?',
                                    parameters: [choice(name: 'Versioning Service', choices: 'no\nyes', description: 'Choose "yes" if you want to release this build')]
                                }
                                if (userChoice == 'no') {
                                    echo "User refuse to release this build, stopping...."
                                }
                            } catch (Exception err) {
                                deploy_status = false
                            }
                        }
                    } catch (Exception err) {
                        deploy_status = false
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
                            version_name = input (
                                 id: 'version', message: 'Input version name', parameters: [
                                    [$class: 'TextParameterDefinition', description: 'Whatever you type here will be your version', name: 'Version']
                                ]
                            )
                        }
                        try {
                            echo "execute command to release service"
                            release_status = true
                        } catch (Exception err) {
                            release_status = false
                        }
                    } catch (Exception err) {
                       release_status = false
                    }
                }
            }
        }

        stage ('Notify') {
            when {
                expression {
                    !build_status || !test_status || !deploy_status || !release_status || deploy_status || release_status
                }
            }
            steps {
                script {
                    if (!build_status) {
                        echo "will notify the user that the build stage is failed"
                    } else if (!test_status) {
                        echo "will notify the user that the test stage is failed"
                    } else if (!deploy_status) {
                        echo "will notify the user that the deploy is failed"
                    } else if (deploy_status) {
                        echo "will notify the user that deploy is success"
                    } else if (!release_status) {
                        echo "will notify the user that release is failed"
                    } else if (release_status) {
                        echo "will notify the user that release is success"
                    }
                }
            }
        }
    }
}