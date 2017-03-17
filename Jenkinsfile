import groovy.json.JsonOutput

def notifyAtomist(buildStatus, endpoint="https://webhook-staging.atomist.services/atomist/jenkins") {

    def gitRemoteUrl = sh(returnStdout: true, script: 'git config --get remote.origin.url').trim()
    def gitCommitSha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()

    def gitBranchName = env.BRANCH_NAME
    if (gitBranchName == null)
        gitBranchName = sh(returnStdout: true, script: 'git name-rev --always --name-only HEAD').trim().replace('remotes/origin/', '')

    def payload = JsonOutput.toJson([
        name: env.JOB_BASE_NAME,
        duration: currentBuild.duration,
        build      : [
            number: env.BUILD_NUMBER,
            status: buildStatus,
            full_url: env.BUILD_URL,
            scm: [
                url: gitRemoteUrl,
                branch: gitBranchName,
                commit: gitCommitSha,
                pr: [
                    number: env.CHANGE_ID,
                    url: env.CHANGE_URL,
                    author: env.CHANGE_AUTHOR,
                    author_email: env.CHANGE_AUTHOR_EMAIL
                ]
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