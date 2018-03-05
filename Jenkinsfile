import groovy.io.FileType

srcDir = '/JSONFiles'

node {
    stage('Checkout'){
      checkout scm
    }
    
    stage('CreateAndUpdateAPI'){
    sh 'git diff --name-only HEAD HEAD~1 > latestChangedFiles.txt'
   
    File file = new File(WORKSPACE+'/latestChangedFiles.txt')
    def lines = file.readLines()
    println "${file} has ${lines.size()} lines of text"
    List files = Arrays.asList(new File(WORKSPACE + srcDir).listFiles())
             for (String item : lines) {
		//env.jsonFileName = item.toString().substring(WORKSPACE.length())
                if(item.toString().startsWith("JSONFiles/")) {
		env.jsonFileName = '/'+item.toString()
		
		
		sh '''echo "**********************************************       Creating clientId and cleintSecret for ADMIN"
		cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
		cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')

		encodeClient="$(echo -n $cid:$cs | base64)"

		echo "**********************************************       Client encoded with code : ${encodeClient}"

		tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
		tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

		echo "**********************************************       Tokens created"

		echo "**********************************************       Comparing to see if WSDL already exists"

		echo "**********************************************       Fetching all APIS"

		apisList=$(curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' )

		createFile=`jq \'{name: .name , context: .context , version: .version}\' ${WORKSPACE}${jsonFileName}`
		newName="$(echo $createFile | jq -r \'.name\')"
		newContext="$(echo $createFile | jq -r \'.context\')"
		newVersion="$(echo $createFile | jq -r \'.version\')"

		echo "**********************************************       CreateFile is ${createFile}"

		match="$(echo $apisList | jq  --arg creName "$newName" --arg creCon "$newContext" --arg creVer "$newVersion"  \'select((.name==$creName) and (.context==$creCon)  and (.version==$creVer))\')"

		if [ -n "$match" ]
		then
		echo "**********************************************       Match found ${match}"
		echo $match | jq \'{id: .id}\' > updateId.json
		updateId="$(echo $match | jq -r \'.id\')"
		echo $updateId
		echo "**************************      EXECUTING UPDATE      ******************************"
		createFileForUpdate=`jq  -s add updateId.json @${WORKSPACE}${jsonFileName}`

		echo $createFileForUpdate > UpdateAPI.json
		echo "**********************************************       Created file UpdateAPI.json to update API"
		curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X PUT -d @UpdateAPI.json https://localhost:9443/api/am/publisher/v0.11/apis/$updateId >>updatedAPI
		echo "**************************      UPDATE COMPLETED      ******************************"
		fi

		if [ -z "$match" ]
		then
		echo "**********************************************       No Match Found"
		echo "**************************      CREATING API      ******************************"
		curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X POST -d @${WORKSPACE}${jsonFileName} https://localhost:9443/api/am/publisher/v0.11/apis >> createdAPI
		echo "**************************      API ${newName} CREATED    ******************************"
		fi
		'''
             }
        }
 
                sh '''echo "**********************************************       Deleting API ${delApi}"    
		cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
		cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')

		encodeClient="$(echo -n $cid:$cs | base64)"
                tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

                delApi=`jq -r \'.id\' deleteAPI.json`

		deleteResp=$(curl -k -H "Authorization: Bearer $tokenCreate" -X DELETE https://localhost:9443/api/am/publisher/v0.11/apis/$delApi)

		if [ -n "$deleteResp" ]
		then
		echo "**********************************************       Error: $deleteResp"
		fi

		if [ -z "$deleteResp" ]
		then
		echo "**********************************************       API $delApi deleted successfully"
		fi

		curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' > FetchedApis.json

		echo "**********************************************       API LIST is written to FetchedApis.json"
                '''

    }
}
