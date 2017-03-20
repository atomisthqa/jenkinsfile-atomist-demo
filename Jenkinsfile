import groovy.json.JsonOutput

/*
 * Retrieve current SCM information from local checkout 
 */
def getSCMInformation() {
    def gitRemoteUrl = sh(returnStdout: true, script: 'git config --get remote.origin.url').trim()
    def gitCommitSha = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()

    // This is defined when using a Multi Pipeline job
    def gitBranchName = env.BRANCH_NAME

    // In single pipeline jobs, we must query the local git metadata
    if (gitBranchName == null)
        gitBranchName = sh(returnStdout: true, script: 'git name-rev --always --name-only HEAD').trim().replace('remotes/origin/', '')

    return gitRemoteUrl, gitBranchName, gitCommitSha
}

/*
 * Notify the Atomist services about the status of a build based from a 
 * git repository.
 */
def notifyAtomist(buildStatus, buildPhase="FINALIZED", 
                  endpoint="https://webhook-staging.atomist.services/atomist/jenkins") {

    scmRemoteURL, branchName, commitSha = getSCMInformation()

    def payload = JsonOutput.toJson([
        name: env.JOB_NAME,
        duration: currentBuild.duration,
        build      : [
            number: env.BUILD_NUMBER,
            phase: buildPhase,
            status: buildStatus,
            full_url: env.BUILD_URL,
            scm: [
                url: scmRemoteURL,
                branch: branchName,
                commit: commitSha,
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
        stage('Build') {
            steps {
                // notifiy Atomist the buid starts now
                // could be at any state but should be done first
                notifyAtomist("UNSTABLE", "STARTED")
                // let's pretend something is being done
                sh "sleep 10"
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