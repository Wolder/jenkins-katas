pipeline {
    agent any
    environment {
        docker_username = "wolder"
    }
    stages {
        stage('clone down') {
            steps {
                stash excludes: '.git', name: 'code'
            }
        }
        stage('Parallel execution') {
            parallel {
                stage('say hello') {
                    steps {
                        sh 'echo "hello world!"'
                    }
                }
                stage('build app') {
                    agent {
                        docker {
                            image 'gradle:jdk11'
                        }
                    }
                    options {
                        skipDefaultCheckout(true)
                    }

                    when {
                        beforeAgent true
                        branch 'master'
                    }

                    steps {
                        unstash 'code'
                        sh 'ci/build-app.sh'
                        archiveArtifacts 'app/build/libs/'
                        sh 'ls'
                        stash excludes: '.git', name: 'code'
                        deleteDir()
                        sh 'ls'
                    }
                }
                stage('test app') {
                    agent {
                        docker {
                            image 'gradle:jdk11'
                        }
                    }
                    options {
                        skipDefaultCheckout(true)
                    }

                    steps {
                        unstash 'code'
                        sh 'ci/unit-test-app.sh' 
                        junit 'app/build/test-results/test/TEST-*.xml'
                    }
                }
            }
        }
        stage('push docker app') {
            environment {
                DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
            }

            steps {
                unstash 'code' //unstash the repository code
                sh 'ci/build-docker.sh'
                sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
                sh 'ci/push-docker.sh'
            }
        }

        stage('component test') {
            when { not { branch "dev/*" } }
            steps {
                unstash 'code' //unstash the repository code
                sh 'ci/component-test.sh'
            }
        }
    }
    post {
        always {
            deleteDir() /* clean up our workspace */
        }
    }
}

