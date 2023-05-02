def SONAR_HOME_DOCKER_VOLUME = 'sonar-home'     // globally shared cache for sonar execution
def OSI_OPENJDK_CONTAINER_ARGUMENTS = "--group-add=999 -v /var/run/docker.sock:/var/run/docker.sock -v ${SONAR_HOME_DOCKER_VOLUME}:/opt/sonar-scanner/.sonar"

pipeline {
    agent none

    options {
        ansiColor('xterm')
        disableConcurrentBuilds()
    }

    stages {
        stage('wake up builder node') {
            agent {
                node {
                    label 'docker && builder'
                }
            }

            options {
                retry(3)
            }

            steps {
                echo 'waking builder node'
            }
        }

        stage('Build, test & publish') {
            agent {
                dockerfile {
                    label 'docker && builder'
                    args "${OSI_OPENJDK_CONTAINER_ARGUMENTS}"
                }
            }

            environment {
                GRADLE_USER_HOME = "$WORKSPACE/.gradle"
                UI_TESTING_BROWSER = "ChromiumHeadlessCI"
            }

            stages {
                stage('Build & Unit tests') {
                    steps {
                        sh './gradlew clean build --warning-mode=none'
                    }

                    post {
                        always {
                            junit allowEmptyResults: true, testResults: '**/build/test-results/test/*.xml'
                        }
                    }
                }

                stage('Integration tests') {
                    steps {
                        sh './gradlew integrationTest --warning-mode=none -x testWebApp -x buildWebApp'
                    }

                    post {
                        always {
                            junit allowEmptyResults: true, testResults: '**/build/test-results/integrationTest/*.xml'
                        }
                    }
                }

                stage('Build images') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                    }

                    options {
                        retry(10)
                    }
                    steps {
                        sh './gradlew buildDockerImage --warning-mode=none'
                    }
                }

                stage('Smoke tests Gleamer') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:gleamerSmokeTest"
                    }
                }

                stage('Smoke tests Milvue') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:sendToMilvueSmokeTest"
                    }
                }

                stage('Smoke tests Icometrix') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:icometrixSmokeTest"
                    }
                }

                stage('Smoke tests Therapixel') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:therapixelSmokeTest"
                    }
                }

                stage('Smoke test Pixyl.NEURO'){
                    environment{
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options{
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:sendToPixylMsSmokeTest"
                        sh "./gradlew --console plain --info :smoke-testing:sendToPixylBvSmokeTest"
                    }

                }

                stage('Smoke tests Transfer study to dicom node') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:transferStudyToDicomNodeSmokeTest"
                    }
                }

                stage('Smoke tests Transfer study to Feops') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:transferStudyToFeopSmokeTest"
                    }
                }

                stage('Smoke tests Transfer study to Cmrad') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:transferStudyToCmradSmokeTest"
                    }
                }

                stage('Smoke tests send to contextflow') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:sendToContextflowSmokeTest"
                    }
                }

                stage('Smoke tests send-study-to-in2bones'){
                     environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                     }

                     options{
                        retry(3)
                     }

                     steps{
                         sh "./gradlew --console plain --info :smoke-testing:sendToIn2BonesFolderTest"
                     }
                }

                stage('Smoke tests SubProcessSmokeTest') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:subProcessSmokeTest"
                    }
                }

                stage('Smoke tests datasetSmokeTest') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                        DOCKER_BUILD_TAG = "${BUILD_TAG}"
                    }

                    options {
                        retry(3)
                    }

                    steps {
                        sh "./gradlew --console plain --info :smoke-testing:datasetSmokeTest"
                    }
                }

                stage('Analysis for pull request') {
                    when {
                        changeRequest()
                    }
                    environment {
                        SONAR_TOKEN = credentials('sonarcloud-token')
                    }
                    steps {
                        sh "./gradlew sonarqube " +
                            "-Dsonar.userHome=/opt/sonar-scanner/.sonar " +
                            "-Dsonar.pullrequest.key=${env.CHANGE_ID} " +
                            "-Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} " +
                            "-Dsonar.pullrequest.base=${env.CHANGE_TARGET}"
                    }
                }
                stage('Analysis for branch') {
                    when {
                        not { changeRequest() }
                    }
                    environment {
                        SONAR_TOKEN = credentials('sonarcloud-token')
                    }
                    steps {
                        sh "./gradlew sonarqube " +
                            "-Dsonar.userHome=/opt/sonar-scanner/.sonar " +
                            "-Dsonar.branch.name=${env.BRANCH_NAME}"
                    }
                }

                stage('Publish images to docker') {
                    environment {
                        DOCKERHUB = credentials('dockerhub-jenkinsOSI')
                    }

                    options {
                        retry(10)
                    }
                    steps {
                        sh './gradlew pushDockerImages cleanDockerImages  -x buildDockerImage  --warning-mode=none'
                    }
                }

                stage('Publish libraries to nexus') {
                    when { not { changeRequest() } }
                    environment {
                        NEXUS = credentials('nexus')
                    }

                    options {
                        retry(5)
                    }
                    steps {
                        sh './gradlew publish -p libs -PnexusUser=$NEXUS_USR -PnexusPassword=$NEXUS_PSW --debug --info'
                    }
                }
            }

            post {
                success {
                    jacoco(
                        execPattern: '**/build/jacoco/*.exec',
                        classPattern: '**/build/classes',
                        sourcePattern: '**/src/main/java',
                        exclusionPattern: '**/test/**,**/integrationTest/**'
                    )
                }
            }
        }
    }
}
