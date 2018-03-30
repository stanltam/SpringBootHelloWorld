import groovy.json.JsonSlurper;
 
//properties([[$class: 'GitLabConnectionProperty', gitLabConnection: 'my-gitlab-connection']])

node{
    stage 'Build, Test and Package'
    env.PATH = "${tool 'M3'}/bin:${env.PATH}"
    checkout scm
    // workaround, taken from https://github.com/jenkinsci/pipeline-examples/blob/master/pipeline-examples/gitcommit/gitcommit.groovy
    def commitid = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def workspacePath = pwd()
    sh "echo ${commitid} > ${workspacePath}/expectedCommitid.txt"
    
   // withMaven(
   //             maven: 'M3',
   //             mavenSettingsConfig: 'a1adf035-653b-410d-b5a6-16b6da77b322',
   //             mavenLocalRepo: '.repository') {
   // 
   //         // Run the maven build
   //         sh "mvn clean package -Dcommitid=${commitid}"
   //     }
        sh "mvn clean package -Dcommitid=${commitid}"
    //sh "mvn clean package -Dcommitid=${commitid}"
}
 
node{
    stage 'Stop, Deploy and Start'
    def workspacePath = pwd()
	sh "echo ${workspacePath}"


    // shutdown
    sh 'curl -X POST http://localhost:8888/shutdown || true'

    // copy file to target location
	sh "mkdir -p ${workspacePath}/deployment"

    sh "cp target/*.jar ${workspacePath}/deployment/"
    // start the application
    sh "chmod 770 ./startApp.sh"

    sh "./startApp.sh"
    // wait for application to respond
  //  sh 'while ! httping -qc1 http://localhost:8888 ; do sleep 1 ; done'
     sh 'netstat -na | grep 8888'
	return 0
}
 
//node{
//    stage 'Smoketest'
//    def workspacePath = pwd()
//    sh "curl --retry-delay 10 --retry 5 http://localhost:8888/info -o ${workspacePath}/info.json"
//    if (deploymentOk()){
//        return 0
//    } else {
//        return 1
//    }
//}
// 
//def deploymentOk(){
//    def workspacePath = pwd()
//    expectedCommitid = new File("${workspacePath}/expectedCommitid.txt").text.trim()
//    actualCommitid = readCommitidFromJson()
//    println "expected commitid from txt: ${expectedCommitid}"
//    println "actual commitid from json: ${actualCommitid}"
//    return expectedCommitid == actualCommitid
//}
// 
//def readCommitidFromJson() {
//    def workspacePath = pwd()
//    def slurper = new JsonSlurper()
//    def json = slurper.parseText(new File("${workspacePath}/info.json").text)
//    def commitid = json.app.commitid
//    return commitid
//}