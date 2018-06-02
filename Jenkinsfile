#!/usr/bin/env groovy
updateGitlabCommitStatus state: 'pending'
pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '20', daysToKeepStr: '10'))
        skipStagesAfterUnstable()
        // Here add name of your saved gitlab configuration with API key
        gitLabConnection('gitlab')
    }
    triggers {
        gitlab(
                triggerOnPush: true,
                triggerOnMergeRequest: true,
                triggerOpenMergeRequestOnPush: "never",
                triggerOnNoteRequest: true,
                noteRegex: "Jenkins please retry a build",
                skipWorkInProgressMergeRequest: false,
                ciSkip: false,
                setBuildDescription: false,
                addNoteOnMergeRequest: false,
                addCiMessage: false,
                addVoteOnMergeRequest: false,
                acceptMergeRequestOnSuccess: false,
                branchFilterType: "NameBasedFilter",
                includeBranchesSpec: "${BRANCH_NAME}",
                secretToken: "abf102fa7f081d4b9727a8c52b88c2ce"
        )
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building....'
                updateGitlabCommitStatus state: 'running'
            }
            post {
                success {
                    updateGitlabCommitStatus name: 'build', state: 'success'
                }
                failure {
                    updateGitlabCommitStatus name: 'build', state: 'failed'
                }
            }
        }
        stage('Test') {
            steps {
                echo 'Testing....'
            }
            post {
                success {
                    updateGitlabCommitStatus name: 'tests', state: 'success'
                }
                failure {
                    updateGitlabCommitStatus name: 'tests', state: 'failed'
                }
            }
        }
    }
    post {
        always {
            echo 'ALWAYS....'
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml,**/target/failsafe-reports/*.xml'
            deleteDir() /* clean up our workspace */
        }
        success {
            // Gitlab notification could go also here (if you don't need to distinguish between build and test phase)
            echo 'SUCCESS....'
            // updateGitlabCommitStatus state: 'success'
        }
        failure {
            // Gitlab notification could go also here (if you don't need to distinguish between build and test phase)
            echo 'FAILURE....'
            // updateGitlabCommitStatus state: 'failed'
        }
        aborted {
            echo 'ABORTED....'
            // updateGitlabCommitStatus state: 'canceled'
        }
    }
}
