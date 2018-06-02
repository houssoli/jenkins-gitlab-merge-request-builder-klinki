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
            // Run the steps in the post section regardless of the completion status of the Pipeline’s or stage’s run.
            echo 'ALWAYS....'

            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml,**/target/failsafe-reports/*.xml'
            deleteDir() /* clean up our workspace */
        }
        changed {
            // Only run the steps in post if the current Pipeline’s or stage’s run has a different completion status from its previous run.
            echo 'CHANGED....'

        }
        success {
            // Only run the steps in post if the current Pipeline’s or stage’s run has a "success" status, typically denoted by blue or green in the web UI.
            echo 'SUCCESS....'

            // Gitlab notification could go also here (if you don't need to distinguish between build and test phase)
            updateGitlabCommitStatus state: 'success'
        }
        fixed {
            // Only run the steps in post if the current Pipeline’s or stage’s run is successful and the previous run failed or was unstable.
            echo 'FIXED....'

        }
        failure {
            // Only run the steps in post if the current Pipeline’s or stage’s run has a "failed" status, typically denoted by red in the web UI.
            echo 'FAILURE....'

            // Gitlab notification could go also here (if you don't need to distinguish between build and test phase)
            updateGitlabCommitStatus state: 'failed'
        }
        unstable {
            // Only run the steps in post if the current Pipeline’s or stage’s run has an "unstable" status, usually caused by test failures, code violations, etc. This is typically denoted by yellow in the web UI.
            echo 'UNSTABLE....'

        }
        aborted {
            // Only run the steps in post if the current Pipeline’s or stage’s run has an "aborted" status, usually due to the Pipeline being manually aborted. This is typically denoted by gray in the web UI.
            echo 'ABORTED....'

            // Gitlab notification could go also here (if you don't need to distinguish between build and test phase)
            updateGitlabCommitStatus state: 'canceled'
        }
        regression {
            // Only run the steps in post if the current Pipeline’s or stage’s run’s status is failure, unstable, or aborted and the previous run was successful.
            echo 'REGRESSION....'

        }
        cleanup {
            echo 'CLEANUP....'
            // Run the steps in this post condition after every other post condition has been evaluated, regardless of the Pipeline or stage’s status.

        }
    }
}
