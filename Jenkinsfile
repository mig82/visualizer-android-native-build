node('android'){
	
	String  visualizerAppName,
			mobileFabricAppName

	String  gitVisualizerAppRepo,
			gitVisualizerAppBranch

	String 	gitCredentialsId,
			mobileFabricCredentialsId

	String 	gitProtocol,
			gitDomain,
			gitOrg,
			gitProject

	stage('Validate inputs'){

		visualizerAppName = VISUALIZER_APP_NAME
		mobileFabricAppName = MOBILE_FABRIC_APP_NAME

		gitVisualizerAppRepo = GIT_VISUALIZER_APP_REPO
		gitVisualizerAppBranch = GIT_VISUALIZER_APP_BRANCH

		gitCredentialsId = GIT_CREDENTIALS
		mobileFabricCredentialsId = MOBILE_FABRIC_CREDENTIALS
		
		//This section parses the git url -e.g.: https://github.com/acme/foo.git
		String[] gitParams = gitVisualizerAppRepo.split('/')
		echo("gitParams class=${gitParams.getClass()}")
		//def gitProtocol = 'https:'
		gitProtocol = gitParams[0]
		echo "gitProtocol=[${gitProtocol}]"
		//gitParams[1] is empty string between '//' after 'https:'
		gitDomain = gitParams[2] //The dns of the git server or 'github.com'.
		gitOrg = gitParams[3] //The git organisation or user -e.g.: 'acme'
		gitProject = gitParams[4].split('\\.')[0] //The name of the project, just before '.git' -e.g.: 'foo'
	}

	stage('init'){
		echo('Building Android phone native')
		if(isUnix()){
			sh("hostname")
			sh("whereis git")
			sh("git --version")
		}
		else{
			bat("hostname")
			bat("where git.exe")
			bat("git --version")
		}
	}
	
	stage('Clone git repo'){
		
		dir("${visualizerAppName}"){
			pwd()
			deleteDir()
		}

		withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: gitCredentialsId, usernameVariable: 'gitUser', passwordVariable: 'gitPassword']]) {		
			//If password contains '@' character it must be encoded to avoid being mistaken by the '@' that separates user:password@url expression.
			String encodedGitPassword = gitPassword.contains("@") ? URLEncoder.encode(gitPassword) : gitPassword
			
			def pwd = pwd()
			echo("Current directory=${pwd}")

			//Clone the repository containting the Visualizer project.
			echo("cloning")
			sh ("git clone ${gitProtocol}//${gitUser}:${encodedGitPassword}@${gitDomain}/${gitOrg}/${gitProject}.git ${visualizerAppName}")
			echo("done cloning")
			if(isUnix()){
				sh("ls -la")
			}
			else{
				bat("dir")
			}
		}
	}
	stage('Clean up'){
		step([$class: 'WsCleanup'])
	}
}
