@Library('jenkins-shared-library') _

/**
* Job Parameters:
*	skipDeploy - whether to deploy build artifacts in case of successful maven build or not (should be false by default)
**/
try {

	def currentVersion
	def revision
	def branch
    def mavenPhase = params.skipDeploy ? "verify" : "deploy"

	slack.notifyBuild()

	node('build-jdk8') {

		stage('Checkout repository') {

			scmVars = checkout scm

			pom = readMavenPom file: 'pom.xml'
			currentVersion = pom.version

			revision = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
			branch = scmVars.GIT_BRANCH.replaceAll("origin/", "")
            
            println("Current version: " + currentVersion)
			println("Revision: " + revision)
			println("Branch: " + branch)

		}

		stage('Build') {

            withMaven(jdk: 'OpenJDK_8', maven: 'Maven_3.6.3', mavenSettingsConfig: custom_maven_settings, options: [artifactsPublisher(disabled: true)],  publisherStrategy: 'EXPLICIT') {
                sh "mvn clean ${mavenPhase} -Dmaven.install.skip=true"
            }

		}

	}

} catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException e) {
	currentBuild.result = "ABORTED"
	throw e
} catch (e) {
	currentBuild.result = "FAILURE"
	throw e
} finally {
	slack.notifyBuild(currentBuild.result)
}
