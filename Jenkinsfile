#!groovy

/*
The MIT License
Copyright (c) 2015-, CloudBees, Inc., and a number of other of contributors
Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.
        THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
*/

node('linux') {


    currentBuild.result = "SUCCESS"
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '2')), disableConcurrentBuilds(), pipelineTriggers([[$class: 'GitHubPushTrigger'], pollSCM('H/15 * * * *')])])

    try {

        stage( 'Checkout' ) {
            checkout scm
            checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], \
            browser: [$class: 'GithubWeb', repoUrl: ''], doGenerateSubmoduleConfigurations: false, \
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'appimage-template']], submoduleCfg: [], \
            userRemoteConfigs: [[url: 'https://github.com/appimage-packages/appimage-template']]]
            checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, \
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'skanlite']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://anongit.kde.org/skanlite']]]
       }
        stage( 'Setup' ) {
            sh 'bundle install'
            def WORKSPACE=pwd()
        }
        stage( 'Build' ) {
            sh 'bundle exec deploy.rb'
       }
       stage('Copy Artifacts') {
          step([$class: 'S3BucketPublisher', dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'ds9-apps', excludedFile: '', flatten: true, gzipFiles: false, keepForever: false, managedArtifacts: true, noUploadOnFailure: true, \
          selectedRegion: 'eu-central-1', showDirectlyInBrowser: true, sourceFile: 'appimage/*', storageClass: 'STANDARD', uploadFromSlave: true, useServerSideEncryption: false]], profileName: 'ds9-apps', userMetadata: []])
       }
       stage('Tests') {
            step([$class: 'LogParserPublisher', failBuildOnError: true, projectRulePath: 'appimage-template/parser.rules', showGraphs: true, unstableOnWarning: true, useProjectRule: true])
      }
   }




    catch (err) {

        currentBuild.result = "FAILURE"

            echo "FAILURE"
        throw err
    }

}
