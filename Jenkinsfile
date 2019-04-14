#!groovy

import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER = env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR = "tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS'
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY

    stage('checkout source code ') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            } else {
                rc = bat returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) {
                error 'hub org authorization failed'
            }

            println(rc)
            if (isUnix()) {
                scratchorg = sh returnStdout: true, script: "sfdx force:org:create -f ./config/project-scratch-def.json -a ci-cd-org -s -w 10 -d 30"
            } else {
                scratchorg = bat returnStdout: true, script: "sfdx force:org:create -f ./config/project-scratch-def.json -a ci-cd-org -s -w 10 -d 30"
            }
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(scratchorg)
            if (robj.status != 0) {
                error 'org creation failed: ' + robj.message
            }
            SFDC_USERNAME = robj.result.username
            println(SFDC_USERNAME)
            robj = null
        }
        stage('Push To Test Org') {
            if (isUnix()) {
                println(' Deploy the code into Scratch ORG.')
                sourcepush = sh returnStdout: true, script: "sfdx force:source:push -u ${SFDC_USERNAME}"
                println(' Assign the Permission Set to the New user ')
                permset = sh returnStdout: true, script: "sfdx force:user:permset:assign -n yeurdreamin -u ${SFDC_USERNAME}"
            } else {
                println(' Deploy the code into Scratch ORG.')
                sourcepush = bat returnStdout: true, script: "sfdx force:source:push -u ${SFDC_USERNAME}"
                println(' Assign the Permission Set to the New user ')
                permset = bat returnStdout: true, script: "sfdx force:user:permset:assign -n yeurdreamin -u ${SFDC_USERNAME}"
            }
            if (sourcepush != 0) {
                error 'push failed'
            }

            if (permset != 0) {
                error 'permission set assignment failed'
            }
        }
        stage('Import Data to test ORG') {
            if (isUnix()) {
                println(' importing data to test org')
                dataimport = sh returnStdout: true, script: "sfdx force:data:tree:import --plan ./data/data-plan.json -u ${SFDC_USERNAME}"
            } else {
                println(' importing data to test org.')
                dataimport = bat returnStdout: true, script: "sfdx force:data:tree:import --plan ./data/data-plan.json -u ${SFDC_USERNAME}"
            }
            println(dataimport)
            if (dataimport != 0) {
                error 'import failed'
            }
        }
        stage('Open test ORG') {
            if (isUnix()) {
                dataimport = sh returnStdout: true, script: "sfdx force:org:open -u ${SFDC_USERNAME}"
            } else {
                dataimport = bat returnStdout: true, script: "sfdx force:org:open -u ${SFDC_USERNAME}"
            }
        }
    }
}
