#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests\\${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST} --setalias ProdSandbox"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST} --setalias ProdSandbox"
            }
            println rc
            if (rc != 0) { error 'hub org authorization failed' }

            // need to pull out assigned username
              if (isUnix()) {
                rmsg = sh returnStdout: true, script: "${toolbelt} force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername -a Jenkins -d 1"
              }else{
                   rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername -a Jenkins -d 1"
              }
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
            def beginIndex = rmsg.indexOf('{')
            def endIndex = rmsg.indexOf('}')
            println(beginIndex)
            println(endIndex)
            def jsobSubstring = rmsg.substring(beginIndex)
            println(jsobSubstring)
            
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(jsobSubstring)
            //if (robj.status != "ok") { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.result.username
            robj = null
            
        }
        
          stage('Push To Scratch Org') {
              if (isUnix()) {
                    rc = sh returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
              }else{
                  rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
              }
            if (rc != 0) {
                error 'push to scratch org failed'
            }
            
          }

          stage('Run Apex Tests') {
            bat script: "mkdir ${RUN_ARTIFACT_DIR}"
            timeout(time: 120, unit: 'SECONDS') {
                rc = bat returnStatus: true, script:"\"${toolbelt}\" force:apex:test:run -c -d ${RUN_ARTIFACT_DIR} -r junit -u ${SFDC_USERNAME} -w 10"
                /*
                if (rc != 0) {
                    error 'apex test run failed'
                } */
            }
        }

        stage('collect results') {
61
            junit keepLongStdio: true, testResults: 'tests\\**\\*-junit.xml'
62
        }


          stage('Delete Scratch Org') {
              if (isUnix()) {
                   rc = sh returnStatus: true, script: "${toolbelt}/ force:org:delete -u Jenkins -p"
              }else{
                  rc = bat returnStatus: true, script: "\"${toolbelt}\" force:org:delete -u Jenkins -p"
              }
            if (rc != 0) {
                error 'Scratch Org Deletion failed'
            }
        }
/*
        stage('Login to Production Sandbox') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} -a TargetSandbox --instanceurl ${SFDC_HOST}"
            if (rc != 0) { error 'Sandbox authorization failed' }
        }
*/        
        stage('Convert to Metadata API format') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:convert -d MDAPI"
        }

        stage('Deploy to Production Sandbox ORG') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:mdapi:deploy -d MDAPI/ -u ProdSandbox -w 100"
        }
             
    }
}
