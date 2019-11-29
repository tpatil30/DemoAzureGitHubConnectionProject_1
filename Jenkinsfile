def pathWorkspace = "E:\\workspace\\DemoApplication_NetFW46"
def machine = "master"
def machineForTests = "master"
def codeEnvironment

def sonarServer = "http://ci.tetrapak.com:9000"
def sonarKey = "DemoApplication_NetFW46"
def projectName = "DemoApplication_NetFW46"


pipeline {
    agent{ 
        node {
             label "${machine}"
             customWorkspace pathWorkspace
        } 
    }
    environment {
        PATHMSDEPLOY = "C:\\Program Files\\IIS\\Microsoft Web Deploy V3"
        PATHMSBUILD = "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\BuildTools\\MSBuild\\15.0\\Bin"
        DEV = "Development"       
        TEST = "Test"
        PROD = "Production"
        NONE = "None"
        DOCK = "Docker"
        DOCKPROD = "DockerProd"
    }

    stages {
        stage ("Checkout") {
            steps {
                script { 
                    codeEnvironment = identifyEnvironment()
                }
                echo "Building Branch: " + env.BRANCH_NAME
                echo "Build Number: " + env.BUILD_NUMBER
                echo "Building Environment: " + codeEnvironment
                echo "Jenkin Project Path: " + pathWorkspace
            }
        }
        
        //stage("Code Analysis"){
        //    when {
        //        expression { codeEnvironment == env.DEV || codeEnvironment == env.TEST || codeEnvironment == env.NONE || codeEnvironment == env.DOCK || codeEnvironment == env.DOCKPROD}
        //    }
        //    steps {
	    //	    echo 'Run SonarScanner and OpenCover'
		//        
	    //    script {
	    //        sqScannerHome = tool 'SonarQube for MSBuild Core';
	    //    }
	
	    //    withSonarQubeEnv('SonarQube') {
                // Deleting the sonar-project.properties file as it causes problems with SonarScanner for MSBuild
	    //        bat "del sonar-project.properties"  
	    //
                // Start SonarScanner
	    //        bat "dotnet \"${sqScannerHome}\\SonarScanner.MSBuild.dll\" begin /k:\"${sonarKey}\" /n:\"${projectName}\" /v:\"1.0\" /d:sonar.cs.opencover.reportsPaths=%CD%\\opencover.xml /d:sonar.host.url=${sonarServer}"

                // Rebuild the application so SonarScanner can do its analysis
        //        bat "\"C:\\Program Files\\dotnet\\dotnet.exe\" build /nodereuse:false"
                
                // Run unit tests and generate code coverage report
	    //        bat 'E:\\OpenCover\\OpenCover.Console.exe -output:"%CD%\\opencover.xml" -register:user -target:"C:\\Program Files\\dotnet\\dotnet.exe" -targetargs:"test" -oldstyle'

                // Stop SonarScanner
        //        bat "dotnet \"${sqScannerHome}\\SonarScanner.MSBuild.dll\" end"
	    //    }
        //  }
        //}	
        stage("Build - Docker") {
            when {
                expression { codeEnvironment == env.DOCK }
                }
            steps {
                echo 'Nuget Package Restore in progress...'
                //bat 'nuget restore'
                
                echo 'Backend Build in progress...'
                dir("\\DemoApplication_NetFW46") {
                    bat 'dotnet publish --configuration Debug /p:EnvironmentName=Docker /p:AngularBuild=builddocker'
                }
            }
		}
        stage("Build - Docker Prod"){
            when{
                expression { codeEnvironment == env.DOCKPROD }
            }
            steps{
                echo 'Nuget Package Restore in progress...'
                //bat 'nuget restore'

                echo 'Backend Build in progress...'
                dir("\\DemoApplication_NetFW46") {
                    bat 'dotnet publish --configuration Release /p:EnvironmentName=DockerProd /p:AngularBuild=builddockerprod'
                }
            }
        }

       stage("Deploy using Docker") {
            when {
                expression { codeEnvironment == env.DOCK }
            }
            steps {
                
                echo 'Copy Dockerfile into Publish folder'
                bat 'copy Dockerfile .\\PublishedDemoApplication_NetFW46'
                
                echo 'Running Docker build and deploy'
                powershell 'E:\\BuildScripts\\dockerBuildAndDeploy.ps1 usdelasdev01.tp1.ad1.tetrapak.com:2375 DemoApplication_NetFW46testcontainer SRVLASDOCKDEV false "E:\\workspace\\DemoApplication_NetFW46\\PublishedDemoApplication_NetFW46" 8186 "1.0"' 
            
            }
		}
        stage("Deploy using Docker on Production"){
            when{
                expression { codeEnvironment == env.DOCKPROD }
            }
            steps{

                timeout(time: 5, unit: "MINUTES") {
                    //wait 5 minutes otherwise abort the process
                    input "Are you sure you want to move to docker's production server?"
                }

                echo 'Copy Dockerfile into Publish folder'
                bat 'copy Dockerfile .\\PublishedDemoApplication_NetFW46'

                echo 'Running Docker build and deploy'
                powershell 'E:\\BuildScripts\\dockerBuildAndDeploy.ps1 usdelascon01.tp1.ad1.tetrapak.com:2375 DemoApplication_NetFW46prodcontainer SRVLASDOCKPROD false "E:\\workspace\\DemoApplication_NetFW46\\PublishedDemoApplication_NetFW46" 8186 "1.0"'



            }
        }
    }
    post {
        success {
            deleteDir()

            script {
           if (codeEnvironment == env.DOCKPROD) {
                 echo 'Record build number in LAPD'
                 bat 'C:\\Windows\\Sysnative\\WindowsPowerShell\\v1.0\\powershell.exe E:\\BuildScripts\\updateVersionInLAPDwithBuildNumber.ps1 -componentName \'AWC - Arganda Weight Control\' -buildNumber '+env.BUILD_NUMBER

                 echo "Sending a message to the Microsoft Teams..."
                sendMessageToTeams("Started ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")

           }
        }

            echo 'Success!'
        }
        failure {
            echo 'Failed!'
        }
    }
    options {
        timeout(time: 60, unit: "MINUTES")
    }
}

// identify which environment will be run from the GIT branch
def identifyEnvironment() {
    def currentBranch = env.BRANCH_NAME.toLowerCase()    
	if (currentBranch.contains("release")) {
        return env.TEST
    }
    else if (currentBranch.contains("develop") || currentBranch.contains("hotfix") ) { 
        return env.DEV
    }
    else if(currentBranch.contains("docker_dev")){
        return env.DOCK
    }
    else if(currentBranch.contains("docker_prod")){
        return env.DOCKPROD
    }
    else { //occurs when the branch is a tag
        return env.PROD
    }
}

def sendMessageToTeams(message) {
     office365ConnectorSend message: "${message}", webhookUrl: "${env.MSTeamsWebHookURL}"
}



