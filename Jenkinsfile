import groovy.io.FileType

srcDir = '/JSONFiles'
apiList = []
jsonFileName = ''

def populateJSONList(path) {
  new File(path + srcDir+"/").eachFile() { file ->
      apiList << file.toString()
      jsonFileName = file.toString().substring(path.length() + srcDir.length())
      println jsonFileName
      println file.toString().substring(path.length() + srcDir.length())
    }
}

node {

    stage('Checkout'){
    checkout scm
    }

    stage('Create API'){
    populateJSONList(WORKSPACE)

    sh "echo $jsonFileName"
    sh "echo ${jsonFileName}"

sh '''
          cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
	  cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')
	  encodeClient="$(echo -n $cid:$cs | base64)"
	  echo ${WORKSPACE}
          xyz="$(echo $jsonFileName)"
	  token=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
	  curl -k -H "Authorization: Bearer $token" -H "Content-Type: application/json" -X POST -d @${WORKSPACE}${jsonFileName} https://localhost:9443/api/am/publisher/v0.11/apis
'''
    }
}
