@Library('jenkins-shared-library') _

/**
* Job Parameters:
*	skipDeploy - whether to deploy build artifacts in case of successful maven build or not (should be false by default)
*	skipJavadoc - whether to build javadoc artifacts as part of the build or not
**/
try {

	def currentVersion
	def revision
	def branch
	def mavenPhase = params.skipDeploy ? "verify" : "deploy"

	slack.notifyBuild()

	node('build-jdk17-isolated') {

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

			withMaven(jdk: 'OpenJDK_17', maven: 'Maven_3.8.4', mavenSettingsConfig: custom_maven_settings, options: [artifactsPublisher(disabled: true)],  publisherStrategy: 'EXPLICIT') {
				if (params.skipJavadoc.toBoolean()) {
					sh "mvn clean ${mavenPhase} -Dmaven.install.skip=true"
				} else {
					sh "mvn clean ${mavenPhase} -Prelease -Dmaven.install.skip=true"
				}
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
