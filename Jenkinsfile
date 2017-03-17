import groovy.json.JsonOutput

def notifyAtomist(buildStatus, endpoint="https://webhook-staging.atomist.services/atomist/jenkins") {

    def gitRemoteUrl = sh(returnStdout: true, script: 'git config --get remote.origin.url').trim()
    def gitCommitSha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def gitBranchName = sh(returnStdout: true, script: 'git name-rev --always --name-only HEAD').trim()

    def payload = JsonOutput.toJson([
        name: env.JOB_NAME,
        build      : [
            number: env.BUILD_NUMBER,
            status: buildStatus,
            full_url: env.BUILD_URL,
            scm: [
                url: gitRemoteUrl,
                branch: env.BRANCH_NAME,
                commit: gitCommitSha
            ]
        ]
    ])
    sh "curl --silent -XPOST -H 'Content-Type: application/json' -d \'${payload}\' ${endpoint}" 
}

pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
                sh 'printenv'
            }
        }
    }
    post { 
        success { 
            notifyAtomist("SUCCESS")
        }
        unstable { 
            notifyAtomist("UNSTABLE")
        }
        failure { 
            notifyAtomist("FAILURE")
        }
    }
}