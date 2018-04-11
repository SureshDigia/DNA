import groovy.io.FileType

srcDir = '/JSONFiles'

node {
    stage('Checkout'){
      checkout scm
    }
    
    stage('CreateAndUpdateAPI'){
             
			env.jsonFileName = '/JSONFiles/'+"${API_NAME}"+'.json'
			def api_status = "${API_STATUS}"
			
                        if( api_status == 'New') {	
				sh '''echo "**********************************************       Creating clientId and cleintSecret for ADMIN"
				cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
				cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')

				encodeClient="$(echo -n $cid:$cs | base64)"

				echo "**********************************************       Client encoded with code : ${encodeClient}"

				tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

				echo "**************************      CREATING API      ******************************"
                                echo "/JSONFiles/${API_NAME}.json"
				curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X POST -d @${WORKSPACE}"/JSONFiles/${API_NAME}.json" https://localhost:9443/api/am/publisher/v0.11/apis
				echo "**************************      API CREATED    ******************************"
				
				echo "**************************      PUBLISHING API      ******************************"
				tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
				tokenPublish=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_publish" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
				apisList=$(curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' )
				createFile=`jq \'{name: .name , context: .context , version: .version}\' ${WORKSPACE}${jsonFileName}`
				newName="$(echo $createFile | jq -r \'.name\')"
				newContext="$(echo $createFile | jq -r \'.context\')"
				newVersion="$(echo $createFile | jq -r \'.version\')"
				match="$(echo $apisList | jq  --arg creName "$newName" --arg creCon "$newContext" --arg creVer "$newVersion"  \'select((.name==$creName) and (.context==$creCon)  and (.version==$creVer))\')"
				apiIdForPublish="$(echo $match | jq -r \'.id\')"
				curl -k -H "Authorization: Bearer $tokenPublish" -X POST "https://localhost:9443/api/am/publisher/v0.11/apis/change-lifecycle?apiId=$apiIdForPublish&action=Publish"
				echo "**************************      API PUBLISHED     ******************************"
				'''
                        }

                        if( api_status == 'Update') {	
				sh '''echo "**********************************************       Creating clientId and cleintSecret for ADMIN"
				cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
				cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')

				encodeClient="$(echo -n $cid:$cs | base64)"

				echo "**********************************************       Client encoded with code : ${encodeClient}"

				tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
				tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

				apisList=$(curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' )

				createFile=`jq \'{name: .name , context: .context , version: .version}\' ${WORKSPACE}${jsonFileName}`
				newName="$(echo $createFile | jq -r \'.name\')"
				newContext="$(echo $createFile | jq -r \'.context\')"
				newVersion="$(echo $createFile | jq -r \'.version\')"

				echo "**********************************************       CreateFile is ${createFile}"

				match="$(echo $apisList | jq  --arg creName "$newName" --arg creCon "$newContext" --arg creVer "$newVersion"  \'select((.name==$creName) and (.context==$creCon)  and (.version==$creVer))\')"

				echo $match | jq \'{id: .id}\' > updateId.json
				updateId="$(echo $match | jq -r \'.id\')"
				echo $updateId
				echo "**************************      EXECUTING UPDATE      ******************************"
				createFileForUpdate=`jq  -s add updateId.json ${WORKSPACE}${jsonFileName}`
				

				echo $createFileForUpdate > UpdateAPI.json
				echo "**********************************************       Created file UpdateAPI.json to update API"
				curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X PUT -d @UpdateAPI.json https://localhost:9443/api/am/publisher/v0.11/apis/$updateId
				echo "**************************      UPDATE COMPLETED      ******************************"
				'''
                        }


                        if( api_status == 'Delete') {	
				sh '''echo "**********************************************       Deleting API ${delApi}"    
				cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
				cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')
				encodeClient="$(echo -n $cid:$cs | base64)"
				tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
				tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
				apisList=$(curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' )

				createFile=`jq \'{name: .name , context: .context , version: .version}\' ${WORKSPACE}${jsonFileName}`
				newName="$(echo $createFile | jq -r \'.name\')"
				newContext="$(echo $createFile | jq -r \'.context\')"
				newVersion="$(echo $createFile | jq -r \'.version\')"

				echo "**********************************************       CreateFile is ${createFile}"

				match="$(echo $apisList | jq  --arg creName "$newName" --arg creCon "$newContext" --arg creVer "$newVersion"  \'select((.name==$creName) and (.context==$creCon)  and (.version==$creVer))\')"
				echo $match | jq \'{id: .id}\' > updateId.json
				deleteId="$(echo $match | jq -r \'.id\')"

				deleteResp=$(curl -k -H "Authorization: Bearer $tokenCreate" -X DELETE https://localhost:9443/api/am/publisher/v0.11/apis/$deleteId)
				if [ -n "$deleteResp" ]
				then
				echo "**********************************************       Error: $deleteResp"
				fi
				if [ -z "$deleteResp" ]
				then
				echo "**********************************************       API $delApi deleted successfully"
				fi
		                '''
                        }


         }
}
