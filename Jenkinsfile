#!groovy

pipeline {
  agent {
    node {
      label 'maven'
    }
  }
  stages {
    rocketSend channel: '#jenkins', message: 'Build Started'
    stage('Build') {
      steps {

        cleanWs()
        git changelog: false, poll: false, url: 'https://github.com/projectethan007/petclinic-new.git'
        sh '''
          set +x
          lastHash=`git log -n 1 --pretty=format:"%H"`
          lastOneHast=`git log -n 1 --skip 1 --pretty=format:"%H"`
          fileChanged=`git diff --name-only $lastHash $lastOneHast`
          message=`git log -1 --pretty=%B`
          author=` git log --format='%an <%ae>' ${lastHash} | head -1`
          echo "Commit Message: ${message}"
          echo "Changed files are: ${fileChanged}"
          echo "Commit Author: ${author}"
          '''
        sh ''' set -x
        mvn clean install -DskipTests -q

        #MSG=`git log -1 --pretty=%B`
        MSG=TEST-25

        echo "$MSG" >> msg.properties
        '''
        archiveArtifacts '**/*'
        rocketSend channel: '#jenkins', message: 'Build Completed'
        script {
          env.JIRA_NO = readFile 'msg.properties'
        }
        step([$class: 'JiraIssueUpdateBuilder', comment: 'Build Completed by jenkins', jqlSearch: "project=TEST and key=${env.JIRA_NO}", workflowActionName: 'BUILD'])
      }
    }
    stage('Unit Test') {
      steps {
        sh '''
        set +x
        mvn clean test -q'''
        script {
          env.JIRA_NO = readFile 'msg.properties'
        }
        sh "echo From Readfile ${env.JIRA_NO}"
        rocketSend channel: '#jenkins', message: 'Unit Test Completed'
        step([$class: 'JiraIssueUpdateBuilder', comment: 'Unit Testing completed by jenkins', jqlSearch: "project=TEST and key=${env.JIRA_NO}", workflowActionName: 'UNIT TESTING'])
      }
    }
    stage('Sonar Analysis') {
      steps {
        withSonarQubeEnv('ADOP Sonar') {
          sh '''
          set +x
          mvn sonar:sonar -q
          echo "Sonar Stat"
          curl --silent -u 42312e909d31c0de2cf84b4b840e8eafe994e5c9: "http://sonar.ethan.svc.cluster.local:9001/sonar/api/resources?resource=org.springframework.samples:spring-petclinic&metrics=lines,blocker_violations,ncloc,classes,directories,files,functions,projects,public_api,statements,branch_coverage"'''
        }
        script {
          env.JIRA_NO = readFile 'msg.properties'
        }
        sh '''
        set +x
        echo From Readfile
        '''
        rocketSend channel: '#jenkins', message: 'Static Code Analysis Completed'
        step([$class: 'JiraIssueUpdateBuilder', comment: 'Code Analysis completed by jenkins', jqlSearch: "project=TEST and key=${env.JIRA_NO}", workflowActionName: 'CODE ANALYSIS'])
      }
    }
    stage('Nexus Upload') {
      steps {
        sh '''
        set +x
        echo "Uploaded Artefact"
        '''

        rocketSend channel: '#jenkins', message: 'Uploaded Artefact to Nexus'
      }
    }
  }
  post {
    always {
      logstashSend failBuild: false, maxLines: -1
    }
  }
}
