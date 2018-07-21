node{
	
	git url: 'https://github.com/ksubbarao/Beach-config.git', branch: 'master'
	
	def pipeline_properties = readProperties file: 'pipeline.properties'
	
	def gitSourceRepo = pipeline_properties.GIT_REPO
	def gitSorceBranch = pipeline_properties.GIT_BRANCH
	
	def mvnHome = tool pipeline_properties.MAVEN_INSTALLATION
	mvnHome = mvnHome + '\\bin'
	
	def winJDK = tool pipeline_properties.JAVA_INSTALLATION
	println mvnHome
	
	def majorVersion = pipeline_properties.MAJOR_VERSION
	def minorVersion = pipeline_properties.MINOR_VERSION
	def patchVersion = pipeline_properties.PATCH_VERSION
	
	env.versionNumber = majorVersion + '.' + minorVersion + '.' + patchVersion + '.' + BUILD_NUMBER
	//env.versionNumber = [majorVersion,'.',minorVersion,'.', patchVersion, '.', BUILD_NUMBER].join()
	
	def server = Artifactory.server 'local_Artifactory'
	
	stage('Checkout'){
		//Checkout Git repo
		git url: gitSourceRepo, branch: gitSorceBranch
		//checkout scm
	}
	
	stage('Code coverage and Analysis'){
		withEnv(["JAVA_HOME=${winJDK}"]){
			bat "${mvnHome}\\mvn sonar:sonar"
		}
	}
	
	stage('Build Automation'){
		
		withEnv(["JAVA_HOME=${winJDK}"]){
		bat "${mvnHome}\\mvn clean package"
		}
	}
	
	stage('Build Management'){
		def buildInfo = Artifactory.newBuildInfo()
		buildInfo.name = 'Beach'
		buildInfo.number = env.versionNumber
		
		def uploadSpec = """{
		"files": [
			{
			  "pattern": "target\\*.war",
			  "target": "maven-dev-local/Web-Apps/Beach/${env.versionNumber}/"
			}
		 ]
		}"""
		
		server.upload(uploadSpec)
		
		//server.upload spec: uploadSpec, buildInfo: buildInfo
		server.publishBuildInfo buildInfo
	}
}
