// Declarative pipelines
pipeline {
    agent any
    
    triggers {
        issueCommentTrigger('.*test this please.*')
    }
    environment {
        GITHUB_TOKEN  = credentials('github-token')
       }
    
    stages {
        // This is a stage.
        stage('Build') {
            steps {
            // Get SHA1 of current commit
            script {
                commit_id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                readme_file = sh(script: "cat README.md", returnStdout: true).trim()
            }
            echo "Fake Build from master with commit id:  ${commit_id}"
            echo " _______________"
            echo "${readme_file}"
            }
        }
        stage('Test') {
            steps {
                sleep 180
            }
        }
    }
    post {
	 success {
	     script {
    	        if (env.CHANGE_ID) {
			echo 'Build succeeded!, alidate it is up to date with origin/master'
			sh(script: "git fetch origin master", returnStdout: true).trim()
			def UpToDate = sh(script: "git merge-base --is-ancestor origin/master ${env.GIT_COMMIT} && echo yes || echo no", returnStdout: true).trim()
			if (UpToDate == 'yes') {
				echo 'We can merge'
				if (pullRequest.mergeable) {
				    pullRequest.labels = ['Jenkins approved']
   				    pullRequest.merge('merge pr to master')
				}
			} else {
				echo 'We need to re-run the test - master has updated since we started the build'
				pullRequest.labels = ['Jenkins disapproved']
				def getPRNum = env.BRANCH_NAME.reverse().take(1).reverse()
				echo "${getPRNum}"
				sh """ curl \
				  -X POST \
				  -H "Accept: application/vnd.github.v3+json" \
				  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
				  https://api.github.com/repos/amit-gueta/exce-mobileye/issues/${getPRNum}/comments \
				  -d '{"body":"test this please"}' 
				   """
			}
		}
	      }
	     cleanWs()
	}
    }
}
