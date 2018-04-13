import groovy.io.FileType
import groovy.json.JsonSlurper
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import hudson.model.* 

node {
    stage('CheckoutPhase'){
      checkout scm
    }

    stage('CreateMetadataPhase'){
      
	String context, versionWithEnv	
	context= "/"+"${API_NAME}"
	pathToTemplate= "${WORKSPACE}" + "/ApiTemplate.json"
	pathToApiMetadata= "${WORKSPACE}" + "/" + "${API_NAME}" + ".json"
	versionWithEnv= "${TARGET_ENV}" + "-" + "${API_VERSION}"
	def inputFile = new File(pathToTemplate) 
	def jsonSlurper = new JsonSlurper()
	def jsonObject = jsonSlurper.parse(inputFile)

	jsonObject.name = "${API_NAME}"
	jsonObject.context = context
	jsonObject.description = "${API_DESCRIPTION}"
	jsonObject.version = versionWithEnv
	jsonObject.wsdlUri= "${WSDL_LOC}"
	
	def endp = jsonSlurper.parseText(jsonObject.endpointConfig)
	endp.production_endpoints.url="${ENDPOINT}"
	String endpointString = JsonOutput.toJson(endp)
	jsonObject.endpointConfig = endpointString	
	new File(pathToApiMetadata).write(new JsonBuilder(jsonObject).toPrettyString())
    }
    
    stage('APIOperationPhase'){
	def api_action = "${ACTION}"

        def props = readJSON file: "${WORKSPACE}"+'/Env.json'
        def envPublish = props['local']

	def newParameter = new StringParameterValue('HOST', envPublish)
	build.replaceAction(new ParametersAction(newParameter))
	println "${HOST}"

        sh "echo $envPublish"

	if( api_action == 'New') {	
		sh '''echo "**********************************************       Creating clientId and cleintSecret for ADMIN"
		echo "$envPublish"

		echo '$envPublish'
		cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://"$envPublish":9443/client-registration/v0.11/register | jq -r \'.clientId\')
		cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://"$envPublish":9443/client-registration/v0.11/register | jq -r \'.clientSecret\')

		encodeClient="$(echo -n $cid:$cs | base64)"
		tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

		echo "**************************      CREATING API      ******************************"
		curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X POST -d @${WORKSPACE}"/JSONFiles/${API_NAME}.json" https://localhost:9443/api/am/publisher/v0.11/apis
		echo "**************************      API CREATED    ******************************"

		echo "**************************      PUBLISHING API      ******************************"
		tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
		tokenPublish=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_publish" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
		apisList=$(curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' )
		createFile=`jq \'{name: .name , context: .context , version: .version}\' ${WORKSPACE}"/JSONFiles/${API_NAME}.json"`
		newName="$(echo $createFile | jq -r \'.name\')"
		newContext="$(echo $createFile | jq -r \'.context\')"
		newVersion="$(echo $createFile | jq -r \'.version\')"
		match="$(echo $apisList | jq  --arg creName "$newName" --arg creCon "$newContext" --arg creVer "$newVersion"  \'select((.name==$creName) and (.context==$creCon)  and (.version==$creVer))\')"
		apiIdForPublish="$(echo $match | jq -r \'.id\')"
		curl -k -H "Authorization: Bearer $tokenPublish" -X POST "https://localhost:9443/api/am/publisher/v0.11/apis/change-lifecycle?apiId=$apiIdForPublish&action=Publish"
		echo "**************************      API PUBLISHED     ******************************"
		'''
          }

        if( api_action == 'Update') {	
		sh '''echo "**********************************************       Creating clientId and cleintSecret for ADMIN"
		cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
		cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')

		encodeClient="$(echo -n $cid:$cs | base64)"
		tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
		tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')

		apisList=$(curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' )

		createFile=`jq \'{name: .name , context: .context , version: .version}\' ${WORKSPACE}"/JSONFiles/${API_NAME}.json"`
		newName="$(echo $createFile | jq -r \'.name\')"
		newContext="$(echo $createFile | jq -r \'.context\')"
		newVersion="$(echo $createFile | jq -r \'.version\')"
		match="$(echo $apisList | jq  --arg creName "$newName" --arg creCon "$newContext" --arg creVer "$newVersion"  \'select((.name==$creName) and (.context==$creCon)  and (.version==$creVer))\')"

		echo $match | jq \'{id: .id}\' > updateId.json
		updateId="$(echo $match | jq -r \'.id\')"
		echo $updateId
		echo "**************************      EXECUTING UPDATE      ******************************"
		createFileForUpdate=`jq  -s add updateId.json ${WORKSPACE}"/JSONFiles/${API_NAME}.json"`
		

		echo $createFileForUpdate > UpdateAPI.json
		echo "**********************************************       Created file UpdateAPI.json to update API"
		curl -k -H "Authorization: Bearer $tokenCreate" -H "Content-Type: application/json" -X PUT -d @UpdateAPI.json https://localhost:9443/api/am/publisher/v0.11/apis/$updateId
		echo "**************************      UPDATE COMPLETED      ******************************"
		'''
        }


        if( api_action == 'Delete') {	
		sh '''echo "**********************************************       Deleting API ${delApi}"    
		cid=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')
		cs=$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientSecret\')
		encodeClient="$(echo -n $cid:$cs | base64)"
		tokenCreate=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_create" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
		tokenView=$(curl -k -d "grant_type=password&username=admin&password=admin&scope=apim:api_view" -H "Authorization: Basic $encodeClient" https://localhost:8243/token | jq -r \'.access_token\')
		apisList=$(curl -k -H "Authorization: Bearer $tokenView" https://localhost:9443/api/am/publisher/v0.11/apis | jq \'.list\' | jq  \'.[] | {id: .id , name: .name , context: .context , version: .version}\' )

		createFile=`jq \'{name: .name , context: .context , version: .version}\' ${WORKSPACE}"/JSONFiles/${API_NAME}.json"`
		newName="$(echo $createFile | jq -r \'.name\')"
		newContext="$(echo $createFile | jq -r \'.context\')"
		newVersion="$(echo $createFile | jq -r \'.version\')"
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
