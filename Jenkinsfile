def image
def imageTag
def version
def fullVersion
def chartVersion

pipeline {
  agent any

  options {
    buildDiscarder(logRotator(daysToKeepStr: '7', artifactDaysToKeepStr: '7'))
  }




  environment {

    REPOSITORY_NAME = "${env.GIT_URL.tokenize('/')[3].split('\\.')[0]}"
    REPOSITORY_OWNER = "${env.GIT_URL.tokenize('/')[2]}"
    GIT_SHORT_REVISION = "${env.GIT_COMMIT[0..7]}"
    DOCKER_SERVER = "${env.MSS_DOCKER_SERVER_PRD}"
    DOCKER_CREDENTIALS = 'mss-docker-registry-prd-credentials'
    DOCKER_ADDR =  'unix:///var/run/docker.sock' 
    IMAGE_NAME = 'ubuntu_test'
    REGISTRY_ADDR = 'my.registry.com'



    SNAPSHOT_REPO = "${env.MSS_MAVEN_SNAPSHOTS_REPO}"
    RELEASE_REPO = "${env.MSS_MAVEN_RELEASES_REPO}"
    
  }

  stages {

    stage('Checkout') {
      steps {
        deleteDir()
        checkout scm
      }
    }
 

    
    
    stage('Compile') {
      steps {
         configFileProvider([configFile(fileId: '58ecf2b0-f51f-4e7e-a89c-68b23919f95b', variable: 'Settings_Maven')]) {
                        retry(count: 3) {
                            rtMavenRun(
                                tool: "Maven 3.8.4", //id specified in Global Tool Configuration
                                pom: 'pom.xml',
                                goals: '-U -s $Settings_Maven clean compile',
                            )
                        }
                    }
                }
            }

    stage('Test') {
      steps {
        script {
          try {
            withMaven(maven: 'maven-3', globalMavenSettingsConfig: 'Settings_Maven', options: [ artifactsPublisher(disabled: true) ]) {
              sh "mvn test"
              //sh "echo test will be done later"
            } 
          } catch (exc) {
            echo 'Tests failed'
            currentBuild.result = 'FAILURE'
          }
        }
      }
    }
  }
}   