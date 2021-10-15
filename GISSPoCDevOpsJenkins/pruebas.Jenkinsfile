def Score

pipeline {
	agent any

	stages {

		stage ('Pruebecillas') {
			steps {
			    node ('UFT_AGENT') {
				
//				echo "Iniciando Pruebecillas"
				
//				script {
				             
//						def kiuwanOutput = readJSON file: "C:/workspaces/DevOpsNat/Jenkins/.jenkins/workspace/DevOps Natural/IC de Natural/kiuwan/output.json"
//						Score = kiuwanOutput.auditResult.score
					runFromAlmBuilder almServerName: 'ALMServer',
						almDomain: 'CCD',
						almProject: 'DEVOPS_PC',
						almUserName: 'JENKINPC',
						almPassword: 'JENKINPC01',
						almRunMode: 'RUN_REMOTE',
						almRunHost: '10.99.104.203',
//						almTestSets: '''Root\\UFT_2021\\Testing_CI_UFT_2021''',
						almTestSets: '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA''',
						almTimeout: '300',
						almClientID: '',
						almRunResultsMode: '',
						almApiKey: '',
						isSSOEnabled: false	
//				}

//				echo "Finalizando Pruebecillas con ${Score}"
				}
			}
		}
	}
}