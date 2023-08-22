pipeline {
    
	agent any
	
	tools {
        maven "maven3"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "44.199.217.199:8081"
        NEXUS_REPOSITORY = "Maven-Demo"
	NEXUS_REPO_ID    = "Maven-Demo"
        NEXUS_CREDENTIAL_ID = credentials('nexuslogin')
	NEXUSIP   = "44.199.217.199"
	NEXUSPORT = "8081"
        NEXUS_USER = "admin"
	NEXUS_PASS = "Password"
        ARTVERSION = "${env.BUILD_ID}"
        RELEASE_REPO = "Maven-Demo"
    }
	
    stages{
        
        stage('BUILD'){
             steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
	    
        stage("list") {
           steps{
		   sh '''
                      ls
		      cd target
	              ls
	           '''
	   }
	}
        

         stage("Publish to Nexus Repository Manager") {
    steps {
        script {
            def pom = readMavenPom(file: "pom.xml")
            def packaging = pom.packaging
            def groupId = pom.groupId
            def artifactId = pom.artifactId
            def version = pom.version

            def filesByGlob = findFiles(glob: "target/*.${packaging}")
            if (!filesByGlob) {
                error "No files matching the specified packaging found"
            }

            def artifactPath = filesByGlob[0].path

            nexusArtifactUploader(
                nexusVersion: NEXUS_VERSION,
                protocol: NEXUS_PROTOCOL,
                nexusUrl: NEXUS_URL,
                groupId: groupId,
                version: version,
                repository: NEXUS_REPOSITORY,
                credentialsId: NEXUS_CREDENTIAL_ID,
                artifacts: [
                    [artifactId: artifactId, classifier: '', file: artifactPath, type: packaging],
                    [artifactId: artifactId, classifier: '', file: "pom.xml", type: "pom"]
                ]
            )
        }
    }
}
        stage('Ansible Deploy to staging'){
            steps{
                ansiblePlaybook([
                inventory       : 'ansible/stage.inventory',
                playbook        : 'ansible/site.yml',
                installation     : 'ansible',
                colorized        : true,
                credentialsId    : 'applogin',
                disableHostKeyChecking: true,
                extraVars: [
                    USER: "admin",
                    PASS: "Password",
                    nexusip: "44.199.217.199",
                    reponame: "Demo",
                    time    : "${env.BUILD_TIMESTAMP}",
                    build: "${env.BUILD_ID}",
                    artifactid: "vprofile",
                    Demo: "vprofile-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"


                ]
             ])
            }
        }


    }


}
