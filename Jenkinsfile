#!/usr/bin/env groovy
@Library('cplib') _

def MASTER_BRANCH = "master"

def PRIVATE_SSH_KEY = ""

def TEMPLATES = ['','ls_etd_sandbox_conductor','ls_nonprod_conductor','ls_prod_conductor']
def STEWARDSHIP = "etd"
def NOTIFICATION = "Elasticsearch-${STEWARDSHIP.toUpperCase()}@bcbsfl.com"
def EXCEPTION_RESPONSE = ""

pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    parameters {
        string (
            name : 'CHANGE_TICKET',
            defaultValue: '',
            description: '[N/A for Non Production Deploy] Change ticket for production deployment (e.g. CRQ####).'
        )
        string (
            name : 'TAS_TICKET',
            defaultValue: '',
            description: '[N/A for Non Production Deploy] Remedy Task (TAS) task ticket for production deployment (e.g. TAS#####).'
        )
        choice(name: 'DEPLOYMENT_TEMPLATE', choices: TEMPLATES)
    }
    stages {
        stage('Validate Prod - Ticket') {
            steps {
                script {
                    if(params.DEPLOYMENT_TEMPLATE == "ls_prod_conductor" && env.BRANCH_NAME == MASTER_BRANCH  ){
                        if(env.CHANGE_TICKET.length() > 0 && env.TAS_TICKET.length() > 0){
                            checkRemedyStatus prod: 'true', notification: "${env.NOTIFICATION}", ticket: "${env.TAS_TICKET}"  
                        } else {
                            currentBuild.result = 'ABORTED'
                            error('Stopping due to lack of change ticket on prod branch')
                            return
                        }
                    }
                }
            }
        }//Close stage 'Validate'
        stage("Deploy"){
            steps {
                script {
                    checkout scm
                    if (params.DEPLOYMENT_TEMPLATE != ""){
                         ansibleTower(
                            towerServer: 'Ansible-Tower-Dev',
                            jobTemplate: params.DEPLOYMENT_TEMPLATE,
                            importTowerLogs: true,
                            removeColor: true,
                            verbose: true
                        )
                    }
                   
                }     
            }
        }
    }
    post {
        failure {
            emailext (
            subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>${EXCEPTION_RESPONSE}</p>
                <p>This is likely result of invalid JSON in stewardship configuration Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
            to: "${NOTIFICATION}"
            )
        }
    }
}
