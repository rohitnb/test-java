pipeline {
  agent any
  stages {
    stage('CodeQL Scan') {
      environment {
        GITHUB_CREDS = credentials('ghes3-jenkins')  //<-- Update the Credential ID here
      }
      steps {
        sh '''
#INITIALISE ENVIRONMENT
CODEQL_PATH="/tmp/codeql-binaries/codeql/codeql" #<---- Modify this
CODEQL_RUNNER_PATH="/tmp/codeql-binaries/codeql-runner-linux" #<---- Modify this

GITHUB_HOST="`echo $GIT_URL | cut -d'/' -f3`"
GITHUB_ORG="`echo $GIT_URL | cut -d'/' -f4`"
GITHUB_REPO="`echo $GIT_URL | cut -d'/' -f5| sed -e 's/\\.git//g'`"

#IDENTIFY THE RIGHT COMMIT AND REF FOR THE WORKFLOW
if [ -z $CHANGE_ID ]
then
      GIT_REF=`git symbolic-ref HEAD`
else
      GIT_REF="refs/pull/$CHANGE_ID/merge"
      GIT_COMMIT=`curl -s --location --request GET "https://$GITHUB_HOST/api/v3/repos/$GITHUB_ORG/$GITHUB_REPO/pulls/$CHANGE_ID" --header 'Accept: application/vnd.github.v3+json' --header "Authorization: token $GITHUB_CREDS_PSW" | jq '.merge_commit_sha' | sed -e 's/\"//g'`
      echo "GIT_COMMIT is $GIT_COMMIT"
fi

#CODEQL SCAN
#1. INIT
$CODEQL_RUNNER_PATH init --repository $GITHUB_ORG/$GITHUB_REPO --github-url https://$GITHUB_HOST --github-auth $GITHUB_CREDS_PSW --codeql-path $CODEQL_PATH

#2. BUILD
#This script is used to "listen" to the build process
chmod +x $WORKSPACE/codeql-runner/codeql-env.sh
. $WORKSPACE/codeql-runner/codeql-env.sh
mvn compile;

#3. ANALYZE
$CODEQL_RUNNER_PATH analyze --repository $GITHUB_ORG/$GITHUB_REPO --github-url https://$GITHUB_HOST --github-auth $GITHUB_CREDS_PSW --commit $GIT_COMMIT --ref $GIT_REF'''
      }
    }
  }
}
