pipeline {
agent any

	environment {
	CPI_TENANT = "${env.MY_CPI_HOST}"
	CPI_CRED = "${env.MY_CPI_CREDS}"	
        IFLOW_ID = "sender2"
        GIT_MAIL = "${env.MY_GIT_MAIL}"
        GIT_USER = "${env.MY_GIT_USER}"
	GIT_URL  = "${env.MY_GIT_URL}"
	REPO_NAME = 'IO-PM'
	GIT_CREDS = "${env.MY_GIT_CREDS}"
    	}

	stages {
		stage('compare versions and store new version') {
		 steps {
		    deleteDir()
		script {
		//download and extract flow from tenant
		    def tempfile=UUID.randomUUID().toString() + ".zip";
                    def response = httpRequest acceptType: 'APPLICATION_ZIP', authentication: "${env.CPI_CRED}" , contentType: 'APPLICATION_ZIP', ignoreSslErrors: true, responseHandle: 'LEAVE_OPEN', timeout: 30,  outputFile: tempfile,url: 'https://' + "${env.CPI_TENANT}" + '/api/v1/IntegrationDesigntimeArtifacts(Id=\''+ "${env.IFLOW_ID}" + '\',Version=\'active\')/$value';
                    def disposition = response.headers.toString();
                    println(disposition);
                    def index=disposition.indexOf('filename')+9;
                    def lastindex=disposition.indexOf('.zip', index);
                    def filename=disposition.substring(index + 1, lastindex + 4);
                    def folder=filename.substring(0, filename.indexOf('.zip'));
                    fileOperations([fileUnZipOperation(filePath: tempfile, targetLocation: folder)])
                    response.close();

		   //remove the zip
                    fileOperations([fileDeleteOperation(excludes: '', includes: tempfile)])
		     		
		  //push the folder content to Git
                    sh 'git config --global user.email ${GIT_MAIL}'
                    sh 'git config --global user.name  ${GIT_USER}'
                    
                    //clone git remote repo
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${env.GIT_CREDS}" ,usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {    
				      sh('git clone https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@' + "${env.MY_GIT_URL}")
                     }
            		//modify local repo
            		fileOperations([fileCopyOperation(excludes: '', flattenFiles: false, includes: folder+"/**", targetLocation: "./" + "${env.REPO_NAME}" + "/IntegrationContent/IntegrationArtefacts/")])
   		   
		    def dirName = "${env.REPO_NAME}"
			
                    //update remote repo
	                dir(dirName) {
    					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${env.GIT_CREDS}" ,usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {    
    						sh 'git commit -am "flow update from CPI pipeline"'
                            			sh('git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@' + "${env.MY_GIT_URL}")
                         }
	                 }
				}
			}
		}
	}
}
