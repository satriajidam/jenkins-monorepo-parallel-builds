#!/usr/bin/env groovy

def CHANGES_LIST = []

pipeline {
  agent {
    node {
      labels '<your_jenkins_agent_label>'
    }
  }
  environment {
    SHORT_COMMIT = "${GIT_COMMIT[0..7]}"
  }
  options {
    timestamps()
    timeout(time: 30, unit: 'MINUTES')
    parallelsAlwaysFailFast()
  }
  stages {
    stage("Detect Changes") {
      steps {
        script {
          currentBuild.displayName = "#$BUILD_NUMBER $JOB_NAME ($SHORT_COMMIT)"
          CHANGES_LIST = detectChanges()
        }
      }
    }
    stage("Perform Builds") {
      steps {
        script {
          doParallelBuild(CHANGES_LIST)
        }
      }
    }
  }
  post {
    always {
      cleanWs()
    }
  }
}

def getAllFolderNames() {
  directoryNameList = []
  rawDirectoryList = sh(script: 'ls -d -1 */', returnStdout: true)
  rawDirectoryList.replaceAll('/','').trim().split("\n").each { directoryName ->
    directoryNameList << directoryName
  }
  return directoryNameList
}

def checkFolderForDiffs(folder) {
  matches = sh(returnStatus:true, script: "git diff --name-only HEAD~1 | grep '${folder}'")
  return matches == 0
}

def detectChanges() {
  changesList = []
  getAllFolderNames().each { folder ->
    if (checkFolderForDiffs(folder)) {
      changesList << folder
      println "Changes detected on ${folder}"
    }
  }
  return changesList
}

def doParallelBuild(CHANGES_LIST) {
  parallelBuildJobs = [:]
  CHANGES_LIST.each { folder ->
    parallelBuildJobs[folder] = generateBuild(folder)
  }
  parallel parallelBuildJobs
}

def generateBuild(folder) {
  return {
    stage("${folder}: Build Stage 1") {
      agent {
        node {
          labels '<your_jenkins_agent_label>'
        }
      }
      environment {
        // Stage 1 environment setup...
      }
      options {
        // Stage 1 build options...
      }
      steps {
        // Stage 1 build steps...
      }
    }
    stage("${folder}: Build Stage N") {
      agent {
        node {
          labels '<your_jenkins_agent_label>'
        }
      }
      environment {
        // Stage N environment setup...
      }
      options {
        // Stage N build options...
      }
      steps {
        // Stage N build steps...
      }
    }
  }
}
