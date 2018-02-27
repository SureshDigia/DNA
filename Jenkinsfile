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
    sh echo $jsonFileName
    sh "echo ${jsonFileName}"
    sh '$(curl -k -X POST -H "Authorization: Basic YWRtaW46YWRtaW4=" -H "Content-Type: application/json" -d @payload.json https://localhost:9443/client-registration/v0.11/register | jq -r \'.clientId\')'
}

