import groovy.io.FileType

srcDir = '/JSONFiles'
def apiList = []

def populateJSONList(path) {
  new File(path + srcDir+"/").eachFileRecurse(FileType.FILES) { file ->
      apiList << file.toString()
      env.jsonFileName = file.toString().substring(path.length())
      println jsonFileName
    }
}

node {

    stage('Checkout'){
    checkout scm
    }

    stage('Create API'){
    populateJSONList(WORKSPACE)

    apiList.each {
        println it
    }
  }
}
