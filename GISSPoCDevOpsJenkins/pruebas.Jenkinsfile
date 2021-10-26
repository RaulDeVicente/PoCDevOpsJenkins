def Score
def ejecutarPruebecillas = false

pipeline {
	agent any

	stages {

		stage ('Pruebecillas2') {
            when {
                expression { !ejecutarPruebecillas }
            }
			steps {
				echo 'Entro en pruebecillas2'
			}
		}

		stage ('Pruebecillas') {
            when {
                expression { ejecutarPruebecillas }
            }
			steps {
				echo 'Entro en pruebecillas'
//				script {
//					def respuestaServico = httpRequest url: 'http://g99dnsa824-ld.portal.ss:15555/ws/giss.ccd.natDevOps.ntdo.tpai.ws:tpaiService/giss_ccd_natDevOps_ntdo_tpai_ws_tpaiService_Port',
//						httpMode: 'POST',
//						customHeaders: [[maskValue: false, name: 'SOAPAction', value: 'giss_ccd_natDevOps_ne@Rtdo_tpai_ws_tpaiService_Binder_pruebaRespuestaServicio']],
//						timeout: 200,
//						requestBody: '''<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://seg-social.es/ccd/tpai/service">
//						   <soapenv:Header/>
//						   <soapenv:Body>
//						      <ser:pruebaRespuestaServicio/>
//						   </soapenv:Body>
//						</soapenv:Envelope>''',
//						consoleLogResponseBody: true,
//						validResponseContent: '<resultado>1</resultado>',
//						wrapAsMultipart: false

//					println('Status: '+ respuestaServico.status)
//					println('Response: '+respuestaServico.content)
//					final int inicio = respuestaServico.content.indexOf('<peticionCambio>') + 16
//					final int fin = respuestaServico.content.indexOf('</peticionCambio>')
//					final peticionCambio = respuestaServico.content.substring(inicio, fin)
//					println('Petición de cambio: '+ peticionCambio)
					
//				}

//			    node ('UFT_AGENT') {
				
//				echo "Iniciando Pruebecillas"
				
				             
//						def kiuwanOutput = readJSON file: "C:/workspaces/DevOpsNat/Jenkins/.jenkins/workspace/DevOps Natural/IC de Natural/kiuwan/output.json"
//						Score = kiuwanOutput.auditResult.score
//					runFromAlmBuilder almServerName: 'ALMServer',
//						almDomain: 'CCD',
//						almProject: 'DEVOPS_PC',
//						almUserName: 'JENKINPC',
//						almPassword: 'JENKINPC01',
//						almRunMode: 'RUN_REMOTE',
//						almRunHost: '10.99.104.203',
//						almTestSets: '''Root\\UFT_2021\\Testing_CI_UFT_2021''',
//						almTestSets: '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA''',
//						almTimeout: '300',
//						almClientID: '',
//						almRunResultsMode: '',
//						almApiKey: '',
//						isSSOEnabled: false	
//				}

//				echo "Finalizando Pruebecillas con ${Score}"
//				}
			}
		}
	}
}