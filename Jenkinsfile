properties([[$class: 'JiraProjectProperty'], parameters([choice(choices: 'develop\nstaging\nmaster', description: 'Select Branch to Build', name: 'Branch')])])
node('docker') {
	stage('Poll') {
		checkout scm
	}
	stage('Build & Unit test'){
		sh 'mvn clean verify -DskipITs=true';
      		junit '**/target/surefire-reports/TEST-*.xml'
      		archiveArtifacts 'target/*.war'
   	}
	stage('Static Code Analysis'){
  		sh 'mvn clean verify sonar:sonar -Dsonar.projectName=Esafe-Project -Dsonar.projectKey=Esafe-Project -Dsonar.projectVersion=$BUILD_NUMBER';
	}
	stage ('Integration Test'){
  		sh 'mvn clean verify -Dsurefire.skip=true';
		junit '**/target/failsafe-reports/TEST-*.xml'
   		archiveArtifacts 'target/*.war'
	}
	stage ('Publish'){
    		def server = Artifactory.server 'Default Artifactory Server'
    		def uploadSpec = """{
    		"files": [
    		{
     		"pattern": "target/Test-0.0.1.war",
     		"target": "Multibranch-pipeline/${BUILD_NUMBER}/",
	 	"props": "Integration-Tested=Yes;Performance-Tested=No"
   		}
           	]
		}"""
		server.upload(uploadSpec)
	}
	stash includes: 'target/Test-0.0.1.war,src/pt/Hello_World_Test_Plan.jmx', name: 'binary'
}
 
