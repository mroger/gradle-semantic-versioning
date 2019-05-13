pipeline {
    agent any
    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                script {
                    env.gitBranch = checkout([
                        $class: 'GitSCM',
                        branches: scm.branches,
                        submoduleCfg: [],
                        userRemoteConfigs: [
                            [url: 'E:/Projects/Spring-boot/semantic-version/.git']
                        ]
                    ]).GIT_BRANCH

                    env.headTag = sh(returnStdout: true, script: 'git describe --all --exact-match `git rev-parse HEAD`').trim()

                    echo env.headTag

                    if (headTag =~ /^tags\/v\d+.\d+.\d+/) {
                        currentBuild.result = 'ABORTED'
                        error 'Ignoring build because there are no new changes'
                    }
                }
            }
        }

        stage('unit tests') {
            steps {
                sh "./gradlew clean test --console=plain"
            }
        }

        stage('build') {
            steps {
                sh "./gradlew -Prelease -q printVersion"
                sh "./gradlew clean build -Prelease --console=plain"
            }
        }

        stage('release') {
            steps {
                sh "git fetch --tags"
                sh "./gradlew -Prelease tag --console=plain"
            }
        }
    }

    post {
        always {
            sh "./gradlew --stop"
        }
    }
}
