// Jenkins declarative pipeline
def projectConfig
def appConfig
def buildManifest
def gitInfo
// def buildVersionNumber // use env.BUILD_VERSION
def defaultBranch = 'develop'
def repoName = env.JOB_BASE_NAME


pipeline {

    agent any

    stages {

        stage('Workspace preparation') {
            steps {
                cleanWs()
                script {
                    //if (binding.hasVariable('sha1')) { branch = sha1 } else { branch = defaultBranch }
                    try { branch = sha1 } catch(Exception) { branch = defaultBranch }
                    gitInfo = checkout poll: false,
                        scm: [
                            $class: 'GitSCM',
                            branches: [[name: branch]],
                            doGenerateSubmoduleConfigurations: false,
                            extensions: [],
                            submoduleCfg: [],
                            userRemoteConfigs: [[
                                credentialsId: 'git-<ORG_NAME>-ci',
                                url: "git@github.com:<ORG_NAME>/${repoName}",
                                refspec: '+refs/pull/*:refs/remotes/origin/pr/*'
                            ]]
                        ]
                    // create short commit
                    gitInfo.GIT_COMMIT_SHORT = gitInfo.GIT_COMMIT.substring(0,7)

                    // set build version
                    if (env.BUILD_VERSION == 'LOCAL') {
                        env.BUILD_VERSION = VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED, "yyMMdd"}${BUILDS_TODAY, XX}.dev.' + gitInfo.GIT_COMMIT_SHORT)
                    }

                    // set docker related vars
                    // env.DOCKER_REGISTRY = '<PRIVATE_DOCKER_REGISTRY_URL>' // use global var
                    env.DOCKER_REPOSITORY = env.JOB_NAME
                    env.DOCKER_TAG = env.BUILD_VERSION
                }

            }
        }

        // 1.1
        stage('Read project properties') {
            steps {
                script {
                    projectConfig = readYaml file: 'project.yml'
                    appConfig = readYaml file: 'appSpec.yml'
                }
            }
        }

        // 2.0
        stage('Static code anaysis and test') {
            steps {
                // 2.1 SonarQube Analysis
                script {
                    def scannerHome = tool env.DEFAULT_SONAR_SCANNER
                    // Write sonar project properties
                    writeFile file: 'sonar-project.properties', text: buildSonarProps(env.JOB_NAME)
                    // Execute  SonarQube
                    withSonarQubeEnv(env.DEFAULT_SONAR_SERVER) {
                        sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                    }
                }

                // 2.2 Code 'linting'
                script {
                    def linterMap = [
                        python: 'edward2a/lint-python',
                        node: 'edward2a/lint-node'
                    ]
                    try {
                        if (appConfig.application.langName in linterMap) {
                            withDockerContainer(args: '-u root', image: linterMap."${appConfig.application.langName}") {
                                sh 'scan src'
                            }
                        } else {
                            echo("INFO: Code linting for ${appConfig.application.langName} is not available.")
                        }
                    } catch(e) {
                        error('Code lint analysis failed')
                        currentBuild.result = 'FAILURE'
                    }
                }

                // 2.3 Unit Testing + coverage
                script {
                    sh 'make unit-tests'
                    sh 'make coverage'
                    cobertura coberturaReportFile: 'output/coverage.xml',
                        failNoReports: true,
                        conditionalCoverageTargets: '70, 0, 0',
                        lineCoverageTargets: '80, 0, 0',
                        maxNumberOfBuilds: 25,
                        methodCoverageTargets: '80, 0, 0',
                        onlyStable: false,
                        zoomCoverageChart: false
                }
            }
        }

/*        // 3.0
        stage('') {
        }
*/

        // 4.0
        stage('Build') {
            steps {
                script {
                    sh 'make build'
                }
            }
        }

        // 5.0
        stage('Artifact packaging and publishing') {
            steps {
                script {
                    buildManifest = readYaml file: 'output/manifest.yml'

                    if (projectConfig.buildOutput == 'container') {
                        if (branch == defaultBranch) {
                            withDockerRegistry([
                                credentialsId: '<ORG_NAME>-dev-docker-registry',
                                url: "https://${env.DOCKER_REGISTRY}"]) {
                                    sh "docker push ${env.DOCKER_REGISTRY}/${env.DOCKER_REPOSITORY}:${env.DOCKER_TAG}"
                            }
                        }
                        // remove local image
                        sh "docker rmi ${env.DOCKER_REGISTRY}/${env.DOCKER_REPOSITORY}:${env.DOCKER_TAG}"
                    }

                    if (branch == defaultBranch) {
                        zip archive: true,
                            dir: 'output',
                            glob: '*',
                            zipFile: "${JOB_NAME}-${BUILD_VERSION}.zip"

                        s3Upload acl:'Private',
                            bucket: '<ORG_NAME>-dev-builds',
                            file: "${JOB_NAME}-${BUILD_VERSION}.zip",
                            path: "${JOB_NAME}/builds/",
                            payloadSigningEnabled: true,
                            sseAlgorithm: 'AES256'
                    }
                }
            }
        }

    }
/*
    post {

        always {
        }

        failure {
        }

        success {
        }

    }
*/
}

// Helper Functions

def buildSonarProps(String projectName) {
    props = [
        'sonar.projectKey=' + projectName.replace('-', '.').replaceAll('/.*$', ''),
        'sonar.projectName=' + projectName,
        'sonar.sources=src'
    ]
    return props.join('\n')
}
