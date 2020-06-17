def server = Artifactory.server 'Artifactory-server'

		 //If artifactory is not defined in Jenkins, then create on:
		// def server = Artifactory.newServer url: 'Artifactory url', username: 'username', password: 'password'

//Create Artifactory Maven Build instance
def rtMaven = Artifactory.newMavenBuild()
//def project_path = "spring-boot-tutorial-basics"
  //jenkins2-course-spring-petclinic
def buildInfo

pipeline {
    agent any

	tools {
		//jdk "Java-1.8"
		maven "Maven-3.6.3"
	}

    stages {
        stage('Clone sources'){
            steps {
                git url: 'https://github.com/nandhakumarr94/JPetclinic'
            }
        }

     	stage('SonarQube analysis') {
	     steps {
		//Prepare SonarQube scanner enviornment
		withSonarQubeEnv('Sonarqube_scanner') {
		   sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar'
		}
	      }
	}


	stage('Artifactory configuration') {
		
	   steps {
		script {
			rtMaven.tool = 'Maven-3.6.3' //Maven tool name specified in Jenkins configuration
		
			rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server //Defining where the build artifacts should be deployed to
			
			rtMaven.resolver releaseRepo:'libs-release', snapshotRepo: 'libs-snapshot', server: server //Defining where Maven Build should download its dependencies from
			
			rtMaven.deployer.artifactDeploymentPatterns.addExclude("pom.xml") //Exclude artifacts from being deployed
			
			//rtMaven.deployer.deployArtifacts =false // Disable artifacts deployment during Maven run
		    
			buildInfo = Artifactory.newBuildInfo() //Publishing build-Info to artifactory
			
			//buildInfo.retention maxBuilds: 10, maxDays: 7, deleteBuildArtifacts: true

			buildInfo.env.capture = true
			}
	    }
	}
	   
	stage('Execute Maven') {
		
		steps {
		   script {
		// dir(project_path) {
		rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
			//}
		}
		
	}}

	stage('Publish build info') {
		steps {
		  script {

		server.publishBuildInfo buildInfo
		}
		}
	}
	stage('status'){
	steps {		emailext (
    subject: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
    body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
    to: "nandhakumarr@live.com"
	) }}
}
}
