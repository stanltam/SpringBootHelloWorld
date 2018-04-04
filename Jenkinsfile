import groovy.json.JsonSlurper;
 
//properties([[$class: 'GitLabConnectionProperty', gitLabConnection: 'my-gitlab-connection']])

node{
    stage '\u2776 Build, Test and Package'
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
	
		mail body: "The job is waiting for your approval, please approve by accessing the following url: http://192.168.15.141:8080/job/SampleSpringBoot/${build_number}/input/002ad408ab77a9fe903f75e8520a3e32/proceedEmpty",
        charset: 'UTF-8',
        from: 'jenkins@gmail.com',
        mimeType: 'text/plain',
        replyTo: 'jenkins@gmail.com',
        subject: "Please approve for the Jenkins Jobs on ${env.JENKINS_URL}",
        to: 'stanltam@gmail.com'
}
 
 	input message: 'Are you confirm to proceed?', submitter: 'stanley'

node{
//    stage '\u2777 Stop, Deploy and Start'
	stage name: 'Stop, Deploy and Start', concurrency: 1
	//For email approval 
	

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
    sh 'while ! httping -qc1 http://localhost:8888 ; do sleep 1 ; done'
}
 
node{
    stage '\u2778 Smoketest'
    def workspacePath = pwd()
    sh "./startApp.sh"
    sh "curl --retry-delay 10 --retry 100 http://localhost:8888/info -o ${workspacePath}/info.json"
    if (deploymentOk()){
		//it is for trying calling other free-style job
		def job = build job: 'say-hello', parameters: [[$class: 'StringParameterValue', name: 'who', value: 'Blog Readers']] 
		echo 'Sending Email....'
        mail bcc: '', body: 'Build Completed successfully!', cc: '', from: '', replyTo: '', subject: '[Jenkin Pipeline]Build Completed', to: 'stanltam@gmail.com'

	  return 0

    } else {
        return 1
    }
}
 
def deploymentOk(){
    def workspacePath = pwd()
    expectedCommitid = new File("${workspacePath}/expectedCommitid.txt").text.trim()
    actualCommitid = readCommitidFromJson()
    println "expected commitid from txt: ${expectedCommitid}"
    println "actual commitid from json: ${actualCommitid}"
    return expectedCommitid == actualCommitid
}
 
def readCommitidFromJson() {
    def workspacePath = pwd()
    def slurper = new JsonSlurper()
    def json = slurper.parseText(new File("${workspacePath}/info.json").text)
    def commitid = json.app.commitid
    return commitid
}