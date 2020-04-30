def builds
builds = {}

if (env.BUILD_VERSION == 'LOCAL') {
    env.BUILD_VERSION = VersionNumber(versionNumberString: '2.0.${BUILD_DATE_FORMATTED, "yyMMdd"}${BUILDS_TODAY, XX}.dev')
}
currentBuild.displayName = "#${env.BUILD_NUMBER} - ${env.BUILD_VERSION}"

pipeline {
    agent any
    stages {
        stage('Prepare Workspace'){
            steps {
                cleanWs()
                echo "======|| BUILD VERSION : ${env.BUILD_VERSION}"
            }
        }
        stage('Build <PROJECT> Components'){
            failFast true
            parallel {
                stage('<PROJ_1>') {
                    steps {
                        script {
                            builds.<PROJ_1> = build job: '<PROJ_1>', parameters: [string(name: 'BUILD_VERSION', value: env.BUILD_VERSION)]
                        }
                    }
                }
                stage('<PROJ_2>') {
                    steps {
                        script {
                            builds.<PROJ_2> = build job: '<PROJ_2>', parameters: [string(name: 'BUILD_VERSION', value: env.BUILD_VERSION)]
                        }
                    }
                }
                /*stage('<PROJ_3>') {
                    steps {
                        script {
                            builds.<PROJ_3> = build job: '<PROJ_3>', parameters: [string(name: 'BUILD_VERSION', value: env.BUILD_VERSION)]
                        }
                    }
                }*/
                stage('<PROJ_4>') {
                    steps {
                        script {
                            builds.<PROJ_4> = build job: '<PROJ_4>', parameters: [string(name: 'BUILD_VERSION', value: env.BUILD_VERSION)]
                        }
                    }
                }
            }
        }
        stage('Prepare Release'){
            steps{
                sh '[ -d output ] || mkdir output'
                getArtifact('<PROJ_1>', builds.<PROJ_1>.getNumber())
                getArtifact('<PROJ_2>', builds.<PROJ_2>.getNumber())
                //getArtifact('<PROJ_3>', builds.<PROJ_3>.getNumber())
                getArtifact('<PROJ_4>', builds.<PROJ_4>.getNumber())
                unzip dir: 'output/<PROJ_1>', glob: '', zipFile: "release/<PROJ_1>-${env.BUILD_VERSION}.zip"
                unzip dir: 'output/<PROJ_2>', glob: '', zipFile: "release/<PROJ_2>-${env.BUILD_VERSION}.zip"
                //unzip dir: 'output/<PROJ_3>', glob: '', zipFile: "release/<PROJ_3>-${env.BUILD_VERSION}.zip"
                unzip dir: 'output/<PROJ_4>', glob: '', zipFile: "release/<PROJ_4>-${env.BUILD_VERSION}.zip"


            }
        }
        stage('Pack <PROJECT> Artifacts'){
            steps{
                zip archive: false,
                    dir: 'output',
                    glob: '*/**',
                    zipFile: "output/<PROJECT>-${BUILD_VERSION}.zip"
            }
        }
        stage('Publish Release'){
            steps {
                archiveArtifacts artifacts: "output/<PROJECT>-${BUILD_VERSION}.zip", onlyIfSuccessful: true
            }
        }
    }
}

def getArtifact(jobName, buildNumber) {
    copyArtifacts selector: specific("${buildNumber}"),
        projectName: jobName,
        filter: '*',
        target: "release",
        excludes: '',
        flatten: false,
        optional: false

}
