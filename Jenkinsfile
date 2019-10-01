node {
    
    def PROJECT_PATH = "atmosphere/spring-boot-samples/spring-boot-sample-atmosphere/"
	
	//Git Cloning
	stage('Git Cloning') { 
	  git "https://github.com/kajal1806/DevOpsMindTree301.git"
	  echo "Clone Completed"
	}
	
    //Maven Building
	stage('Maven Build') {
	    dir(PROJECT_PATH) {
	        sh "mvn clean install"
		    echo "Maven build completed"   
	    }
	}
	
    //Sonar Analysis
    stage('SonarQube Analysis') { 
        dir(PROJECT_PATH) {
            withSonarQubeEnv("SonarQubeServer") {
                sh "mvn sonar:sonar"
                echo "Sonar Analysis Completed"
            }    
        } 
    }	
	
    //Artifactory Upload
	stage('Artifactory Upload') {
		def server = Artifactory.server("ArtifactoryServer")
		def uploadSpec = """{		
			"files":[
			{
				"pattern":"/root/.jenkins/workspace/pipeline-project/${PROJECT_PATH}/target/*.jar",
				"target":"jenkins/atmosphere/Build #${env.BUILD_NUMBER}"
			}
			]
			}"""
		server.upload(uploadSpec)
        echo "Artifacts Uploaded"
	}
	
	//Kill particular containers
	stage("Container Prune") {
		try {
    		sh "docker container prune -f"
    		sh "docker kill atmosphereimage"
    		sh "docker rm atmosphereimage"
    		echo "Containers killed"
		} catch (err) {
		    echo "No matching Containers available to kill"
		}
	}
	
	//Build Docker Images
	stage('Docker Image Build') {
		sh "docker build -t atmosphereimage ${PROJECT_PATH}"
		echo "Image build complete"
	}
	
    //Spinning up the Containers
    stage('Spin Up Containers') {
        try {
            sh "docker kill appcontainer"
            sh "docker rm appcontainer"
            sh "docker run -d --name appcontainer -p 8089:8080 atmosphereimage"
            echo "Container is up and running"    
        } catch (err) {
            echo "No such container available"
            sh "docker run -d --name appcontainer -p 8089:8080 atmosphereimage"
        }
    }
}
