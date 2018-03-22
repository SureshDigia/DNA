import groovy.io.FileType

srcDir = '/JSONFiles'

node {
    stage('Checkout'){
      checkout scm
    }
    
    stage('CreateAndUpdateAPI'){
             
			env.jsonFileName = "${API_NAME}"+'.json'
			def api_status = "${API_STATUS}"
			
                        if( api_status == 'New') {	
			sh '''echo "**********************************************       Creating clientId and cleintSecret for ADMIN"
			cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
			cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')

			encodeClient="$(echo -n $cid:$cs | base64)"

			echo "**********************************************       Client encoded with code : ${encodeClient}"

			tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
			tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

			echo "**************************      CREATING API      ******************************"
			curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X POST -d @${WORKSPACE}${jsonFileName} https://localhost:9443/api/am/publisher/v0.11/apis >> createdAPI
			echo "**************************      API ${newName} CREATED    ******************************"
			'''
                        }





                
		if(lines.contains("deleteAPI.json")) {
		        sh '''echo "**********************************************       Deleting API ${delApi}"    
			cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
			cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')
			encodeClient="$(echo -n $cid:$cs | base64)"
		        tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
			tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
		        delApi=`jq -r \'.id\' deleteAPI.json`
			deleteResp=$(curl -k -H "Authorization: Bearer $tokenCreatel" -X DELETE https://localhost:9443/api/am/publisher/v0.11/apis/$delApi)
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

                        sh '''echo "**********************************************       Fetching all APIs"    
			cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
			cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')
			encodeClient="$(echo -n $cid:$cs | base64)"
			tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
                        curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' > FetchedApis.json
			echo "**********************************************       API LIST is written to FetchedApis.json"

                        
			git remote set-url origin git@github.com:SureshDigia/DNA.git
			git checkout master
                        git pull			
			git status                        
			git add FetchedApis.json latestChangedFiles.txt
                        git commit -m 'Commit FetchedApis.json and latestChangedFiles.txt file.'
                        git push origin HEAD:master			
                        '''


         }
}
