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
			tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

			echo "**************************      CREATING API      ******************************"
			curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X POST -d @${WORKSPACE}${jsonFileName} https://localhost:9443/api/am/publisher/v0.11/apis >> createdAPI
			echo "**************************      API CREATED    ******************************"
			'''
                        }

         }
}
