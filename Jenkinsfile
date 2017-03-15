import groovy.json.JsonOutput

def notifyAtomist(buildStatus, endpoint="https://webhook-staging.atomist.services/atomist/jenkins") {

    def gitRemoteUrl = sh(returnStdout: true, script: 'git config --get remote.origin.url').trim()
    def gitCommitSha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def gitBranchName = sh(returnStdout: true, script: 'git name-rev --always --name-only HEAD').trim()

    def payload = JsonOutput.toJson([
        build      : [
            number: env.BUILD_ID,
            status: buildStatus,
            full_url: env.BUILD_URL,
            scm: [
                url: gitRemoteUrl,
                branch: gitBranchName,
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