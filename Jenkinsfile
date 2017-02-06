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

	def jenkinsWorkspace

	String 	hBuildGlobalProps,
			visualizerHome,
			equinoxJar,
			imageMagicHome,
			androidHome

	//TODO: Remove this temporary fix. Needed because git is not on the PATH for the windows slave we have. Env var GIT_HOME is defined at windows node level.
	String gitHome 
	stage('Patch windows git'){
		gitHome = GIT_HOME?GIT_HOME:""
	}

	stage('Validate inputs'){

		visualizerAppName = VISUALIZER_APP_NAME
		mobileFabricAppName = MOBILE_FABRIC_APP_NAME

		gitVisualizerAppRepo = GIT_VISUALIZER_APP_REPO
		gitVisualizerAppBranch = GIT_VISUALIZER_APP_BRANCH

		gitCredentialsId = GIT_CREDENTIALS
		mobileFabricCredentialsId = MOBILE_FABRIC_CREDENTIALS
	}

	stage('Prepare Git variables'){	
		//This section parses the git url -e.g.: https://github.com/acme/foo.git
		String[] gitParams = gitVisualizerAppRepo.split('/')
		gitProtocol = gitParams[0] // 'https:'|'ssh:'
		echo "gitProtocol=[${gitProtocol}]"
		//gitParams[1] is empty string between '//' after 'https:'
		gitDomain = gitParams[2] //The dns of the git server or 'github.com'.
		gitOrg = gitParams[3] //The git organisation or user -e.g.: 'acme'
		gitProject = gitParams[4].split('\\.')[0] //The name of the project, just before '.git' -e.g.: 'foo'
	}
	
	stage('Prepare Visualizer variables'){
		hBuildGlobalProps = "HeadlessBuild-Global.properties"

		//These env vars are set for each node.
		visualizerHome = VISUALIZER_HOME
		imageMagicHome = IMAGEMAGIC_HOME
		androidHome = ANDROID_HOME

		//These env vars are set globally.
		equinoxJar = EQUINOX_JAR
	}

	stage('init'){
		echo('Building Android phone native')

		jenkinsWorkspace = pwd() 

		if(isUnix()){
			sh("hostname")
			sh("whereis git")
			sh("git --version")
		}
		else{
			bat("hostname")
			//bat("where git.exe") //Can't do this until git gets added to the Windows box PATH.
			bat("${gitHome}git --version")
		}
	}


	stage('Prepare global headless build properties'){

		node('nix'){

				//Create ${hBuildGlobalProps} from scratch.
				sh("rm -f ${hBuildGlobalProps}")
				sh("echo 'workspace.location=${jenkinsWorkspace}' >> ${hBuildGlobalProps}")
				sh("echo 'eclipse.equinox.path=${visualizerHome}/plugins/${equinoxJar}' >> ${hBuildGlobalProps}")
				sh("echo 'imagemagic.home=${imageMagicHome}' >> ${hBuildGlobalProps}")
				sh("echo 'android.home=${androidHome}' >> ${hBuildGlobalProps}")

				//Writes blank BlackBerry settings into the ${hBuildGlobalProps} just in case Visualiser validates they have to be there.
				sh("echo 'bb10.ndk.home=' >> ${hBuildGlobalProps}")
				sh("echo 'bb10.signing.keys.home=' >> ${hBuildGlobalProps}")
				sh("echo 'bb10.emulator.ip=' >> ${hBuildGlobalProps}")
				sh("echo 'bb10.emulator.password=' >> ${hBuildGlobalProps}")
				sh("echo 'bb10.vmware.home=' >> ${hBuildGlobalProps}")
				sh("cat ${hBuildGlobalProps}")

				stash (includes: "${hBuildGlobalProps}", name: "BUILD_GLOBAL_PROPS")
		}
		unstash("BUILD_GLOBAL_PROPS")
		if(isUnix()){
			sh("ls -la")
			sh("cat ${hBuildGlobalProps}")
		}
		else{
			bat("dir")
			bat("more ${hBuildGlobalProps}")
		}
	}
	
	stage('Clone git repo'){
		
		//Remove the Visualizer project's root directory left here by the previous build.
		isUnix()?sh("rm -rf ${visualizerAppName}"):bat("if exist {visualizerAppName} rd /q /s ${visualizerAppName}")

		withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: gitCredentialsId, usernameVariable: 'gitUser', passwordVariable: 'gitPassword']]) {		
			//If password contains '@' character it must be encoded to avoid being mistaken by the '@' that separates user:password@url expression.
			String encodedGitPassword = gitPassword.contains("@") ? URLEncoder.encode(gitPassword) : gitPassword
			
			def pwd = pwd()
			echo("Current dir=${pwd}")

			//Clone the repository containting the Visualizer project.
			String cloneCmd = "${gitHome}git clone ${gitProtocol}//${gitUser}:${encodedGitPassword}@${gitDomain}/${gitOrg}/${gitProject}.git ${visualizerAppName}"
			isUnix()?sh(cloneCmd):bat(cloneCmd)
			
			isUnix()?sh("ls -la"):bat("dir")
		}
	}
	stage('Clean up'){
		deleteDir()
	}
}
