import groovy.io.FileType

srcDir = '/JSONFiles'

node {
    stage('Checkout'){
      checkout scm
    }
    
    stage('Create API'){

    sh 'git diff --name-only HEAD HEAD~1 > latestChangeFiles.txt'
   
    
    List files = Arrays.asList(new File(WORKSPACE + srcDir).listFiles())
    for (String item : files) {
		env.jsonFileName = item.toString().substring(WORKSPACE.length())
		
		sh '''#!/bin/bash
		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       Creating clientId and clientSecret for ADMIN"
		cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
		cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')

		encodeClient="$(echo -n $cid:$cs | base64)"

		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       Client encoded with code : ${encodeClient}"

		token=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       Token created : ${token}"

		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       Comparing to see if WSDL already exists"

		databasename=`jq \'{id: .id , name: .name, wsdl: .wsdlUri}\' createOutput.json`
		`jq \'{id: .id , name: .name, wsdl: .wsdlUri}\' createOutput.json > userList.json`
		createFile=`jq -r \'.wsdlUri\' ${WORKSPACE}${jsonFileName}`

		match="$(echo $databasename | jq  --arg creFile "$createFile" \'select(.wsdl==$creFile)\')"

		if [ -z "$match" ]
		then
		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       No Match Found"
		echo "**************************CREATING API******************************"
		curl -k -H "Authorization: Bearer $token" -H "Content-Type: application/json" -X POST -d @${WORKSPACE}${jsonFileName} https://localhost:9443/api/am/publisher/v0.11/apis >> createOutput.json
		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       Final response is written to createOutput.json file"
		fi

		if [ -n "$match" ]
		then
		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       Match found"
		echo $match | jq \'{id: .id}\' > updateId.json
		wsdl="$(echo $match | jq -r \'.wsdl\')"
		updateId= "$(echo $match | jq -r \'.id\')"
		echo $updateId
		echo "**************************UPDATE EXECUTING******************************"
		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       Matched WSDL is ${wsdl}"
		createFileForUpdate=`jq  -s add updateId.json createDEMO.json`

		echo $createFileForUpdate > UpdateAPI.json
		echo "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!       Created file UpdateAPI.json to update API"
		curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X PUT -d @UpdateAPI.json https://localhost:9443/api/am/publisher/v0.11/apis/$updateId
		fi'''
    }
  }
}
