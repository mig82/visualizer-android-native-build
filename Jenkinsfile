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

		if(isUnix()){
			visualizerHome = MAC_VISUALIZER_HOME
			imageMagicHome = MAC_IMAGEMAGIC_HOME
			androidHome = MAC_ANDROID_HOME
		}
		else{
			visualizerHome = WIN_VISUALIZER_HOME
			imageMagicHome = WIN_IMAGEMAGIC_HOME
			androidHome = WIN_ANDROID_HOME
		}

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
			bat("where git.exe")
			bat("git --version")
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

				stash (includes: './${hBuildGlobalProps}', name: 'BUILD_GLOBAL_PROPS')
		}
		unstash('BUILD_GLOBAL_PROPS')
		if(isUnix()){
			sh("ls -la")
			sh("cat ./${hBuildGlobalProps}")
		}
		else{
			bat("dir")
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
		deleteDir()
	}
}
