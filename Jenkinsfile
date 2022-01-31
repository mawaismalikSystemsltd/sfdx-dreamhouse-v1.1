#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG = "mawais.malik@empathetic-unicorn-9xl702.com" //env.HUB_ORG_DH
    def SFDC_HOST = "https://login.salesforce.com" //env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = "JWT_KEY_FILE" //env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY = "3MVG9pRzvMkjMb6ldL9vVDPBGsdJIUFY.DLZsh8lJcWBI_V2L5O6HI.tjFmCt2zOWwKVXcZTlWPsKarOJmKto" //env.CONNECTED_APP_CONSUMER_KEY_DH

    
    println 'KEY IS'
    println JWT_KEY_CRED_ID
    println 'ORG IS'
    println HUB_ORG
    println 'HOST IS'
    println SFDC_HOST
    println 'APP_KEY IS'
    println CONNECTED_APP_CONSUMER_KEY
    
    
    
    //def sfdxTool
    def toolbelt = tool 'sfdx'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }
//    withEnv(["HOME=${env.WORKSPACE}"]) {
//        withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        
        
        
 //       stage('Create Scratch Org') {

  //          rc = sh returnStatus: true, script: "${sfdxTool}/sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY}  --jwtkeyfile ${jwt_key_file} --username ${HUB_ORG} --instanceurl ${SFDC_HOST}  --setdefaultdevhubusername"
 //           if (rc != 0) { error 'hub org authorization failed' }

            // need to pull out assigned username
 //           rmsg = sh returnStdout: true, script: "${sfdxTool}/sfdx force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername"
 //           printf rmsg
 //           def jsonSlurper = new JsonSlurperClassic()
 //           def robj = jsonSlurper.parseText(rmsg)
 //           if (robj.status != "ok") { error 'org creation failed: ' + robj.message }
//            SFDC_USERNAME=robj.username
//            robj = null

//        }

//        stage('Push To Test Org') {
//            rc = sh returnStatus: true, script: "${sfdxTool}/sfdx force:source:push --targetusername ${SFDC_USERNAME}"
//            if (rc != 0) {
//                error 'push failed'
//            }
            // assign permset
//            rc = sh returnStatus: true, script: "${sfdxTool}/sfdx force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname DreamHouse"
//            if (rc != 0) {
//                error 'permset:assign failed'
//            }
//        }

//        stage('Run Apex Test') {
//            sh "mkdir -p ${RUN_ARTIFACT_DIR}"
//            timeout(time: 120, unit: 'SECONDS') {
//                rc = sh returnStatus: true, script: "${sfdxTool}/sfdx force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
//                if (rc != 0) {
//                    error 'apex test run failed'
//                }
//            }
//        }
//        }     

//        stage('collect results') {
//            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
//        }
//    }
    
    
     withEnv(["HOME=${env.WORKSPACE}"]) {
        
        withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'server_key_file')]) {

            // -------------------------------------------------------------------------
            // Authorize the Dev Hub org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize DevHub') {
                rc = command "${toolbelt}/sfdx auth:jwt:grant --instanceurl ${SFDC_HOST} --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername --setalias HubOrg"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Create new scratch org to test your code.
            // -------------------------------------------------------------------------

            stage('Create Test Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ciorg --wait 10 --durationdays 1"
                if (rc != 0) {
                    error 'Salesforce test scratch org creation failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Display test scratch org info.
            // -------------------------------------------------------------------------

            stage('Display Test Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:display --targetusername ciorg"
                if (rc != 0) {
                    error 'Salesforce test scratch org display failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Push source to test scratch org.
            // -------------------------------------------------------------------------

            stage('Push To Test Scratch Org') {
                rc = command "${toolbelt}/sfdx force:source:push --targetusername ciorg"
                if (rc != 0) {
                    error 'Salesforce push to test scratch org failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Run unit tests in test scratch org.
            // -------------------------------------------------------------------------

            stage('Run Tests In Test Scratch Org') {
                rc = command "${toolbelt}/sfdx force:apex:test:run --targetusername ciorg --wait 10 --resultformat tap --codecoverage --testlevel ${TEST_LEVEL}"
                if (rc != 0) {
                    error 'Salesforce unit test run in test scratch org failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Delete test scratch org.
            // -------------------------------------------------------------------------

            stage('Delete Test Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:delete --targetusername ciorg --noprompt"
                if (rc != 0) {
                    error 'Salesforce test scratch org deletion failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Create package version.
            // -------------------------------------------------------------------------

            stage('Create Package Version') {
                if (isUnix()) {
                    output = sh returnStdout: true, script: "${toolbelt}/sfdx force:package:version:create --package ${PACKAGE_NAME} --installationkeybypass --wait 10 --json --targetdevhubusername HubOrg"
                } else {
                    output = bat(returnStdout: true, script: "${toolbelt}/sfdx force:package:version:create --package ${PACKAGE_NAME} --installationkeybypass --wait 10 --json --targetdevhubusername HubOrg").trim()
                    output = output.readLines().drop(1).join(" ")
                }

                // Wait 5 minutes for package replication.
                sleep 300

                def jsonSlurper = new JsonSlurperClassic()
                def response = jsonSlurper.parseText(output)

                PACKAGE_VERSION = response.result.SubscriberPackageVersionId

                response = null

                echo ${PACKAGE_VERSION}
            }


            // -------------------------------------------------------------------------
            // Create new scratch org to install package to.
            // -------------------------------------------------------------------------

            stage('Create Package Install Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias installorg --wait 10 --durationdays 1"
                if (rc != 0) {
                    error 'Salesforce package install scratch org creation failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Display install scratch org info.
            // -------------------------------------------------------------------------

            stage('Display Install Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:display --targetusername installorg"
                if (rc != 0) {
                    error 'Salesforce install scratch org display failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Install package in scratch org.
            // -------------------------------------------------------------------------

            stage('Install Package In Scratch Org') {
                rc = command "${toolbelt}/sfdx force:package:install --package ${PACKAGE_VERSION} --targetusername installorg --wait 10"
                if (rc != 0) {
                    error 'Salesforce package install failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Run unit tests in package install scratch org.
            // -------------------------------------------------------------------------

            stage('Run Tests In Package Install Scratch Org') {
                rc = command "${toolbelt}/sfdx force:apex:test:run --targetusername installorg --resultformat tap --codecoverage --testlevel ${TEST_LEVEL} --wait 10"
                if (rc != 0) {
                    error 'Salesforce unit test run in pacakge install scratch org failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Delete package install scratch org.
            // -------------------------------------------------------------------------

            stage('Delete Package Install Scratch Org') {
                rc = command "${toolbelt}/sfdx force:org:delete --targetusername installorg --noprompt"
                if (rc != 0) {
                    error 'Salesforce package install scratch org deletion failed.'
                }
            }
        }
    }

}
def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
        return bat(returnStatus: true, script: script);
    }
}
    
    

