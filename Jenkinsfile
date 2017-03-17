import groovy.json.JsonOutput

def notifyAtomist(buildStatus, endpoint="https://webhook-staging.atomist.services/atomist/jenkins") {

    //def gitRemoteUrl = sh(returnStdout: true, script: 'git config --get remote.origin.url').trim()
    //def gitCommitSha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    //def gitBranchName = sh(returnStdout: true, script: 'git name-rev --always --name-only HEAD').trim()

    def payload = JsonOutput.toJson([
        name: env.JOB_BASE_NAME,
        duration: 
        build      : [
            number: env.BUILD_NUMBER,
            status: currentBuild.result,
            full_url: env.BUILD_URL,
            scm: [
                url: env.GIT_URL,
                branch: env.BRANCH_NAME,
                commit: env.GIT_COMMIT,
                pr: [
                    number: env.CHANGE_ID,
                    url: env.CHANGE_URL,
                    author: env.CHANGE_AUTHOR,
                    author_email: CHANGE_AUTHOR_EMAIL
                ]
            ],
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