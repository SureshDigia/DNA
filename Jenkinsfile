import groovy.io.FileType

srcDir = '/JSONFiles'
apiList = []

def populateJSONList(path) {
  new File(path + srcDir+"/").eachFile() { file ->
      apiList << file.toString()
      env.jsonFileName = file.toString().substring(path.length())
      println jsonFileNamel
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
