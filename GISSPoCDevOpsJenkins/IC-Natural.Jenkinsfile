// Pipeline de la PoC del IC de las aplicaciones Natural.

// Constantes:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'
// Variable con la ubicaci�n de las librer�as necesarias para realizar el deploy de Natural.
def libreriasDeploy = 'C:/workspaces/DevOpsNat/NO4Jenkins/deploy'
// Variable con la URL de webMethods.
def urlWebMethods = 'http://g99dnsa824-ld.portal.ss:15555'


// Variables que definen los datos del proyecto/aplicaci�n
def gitRepositorio = 'PoCNatDevOps'
def naturalProyecto = 'GISSPoCNatDevOps'
def uftDominio = 'CCD'
def uftProyecto = 'DEVOPS_PC'
def uftUsuario = 'JENKINPC'
def uftPassword = 'JENKINPC01'
def uftTestSets = '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA'''


// Variables que se calculan en el Pipe.
// Variable con la puntuaci�n obtenida en Kiuwan.
def KiuwanScore
// Variable con la petici�n de cambio generada por TPAI.
def TpaiPeticionCambio


// Variables para decidir qu� stage se ejecutan
def ejecutarCheckout = true
def ejecutarKiuwan = true
def ejecutarDeploy = true
def ejecutarTPAI = true
def ejecutarUnitTest = false
def ejecutarUFT = false


pipeline {
	agent any

	tools {
	   jdk "Java"
	}

	stages {

		stage('CheckOut Natural Git') {
			when {
				expression { ejecutarCheckout }
			}
			steps {
				echo "Iniciando CheckOut de Git"

				// Obtiene el c�digo del GitHub repository
				// git branch:'main', url: 'https://github.com/RaulDeVicente/PoCNatDevOps.git'
				checkout([$class: 'GitSCM',
					branches: [[name: '*/main']],
					extensions: [[$class: 'RelativeTargetDirectory',
					relativeTargetDir: "${naturalProyecto}"]],
					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])

				echo "Finalizando CheckOut de Git"
			}
		}

		stage('An�lisis de c�digo (Kiuwan)') {
			when {
				expression { ejecutarKiuwan }
			}
			steps {
				echo "Iniciando An�lisis de c�digo (Kiuwan)"

				script {

					kiuwan connectionProfileUuid: 'pqvj-J6Ik',
						applicationName_dm: "${naturalProyecto}",
						selectedMode: 'DELIVERY_MODE',
						sourcePath: "${naturalProyecto}/${naturalProyecto}/Natural-Libraries",
						indicateLanguages_dm: true,
						languages_dm: 'natural',
						timeout_dm: 30,
						waitForAuditResults_dm: true

					def kiuwanOutput = readJSON file: "${env.WORKSPACE}/kiuwan/output.json"
					KiuwanScore = kiuwanOutput.auditResult.score
				}

				echo "Finalizando An�lisis de c�digo (Kiuwan) con Score: ${KiuwanScore}"
			}
		}

		stage('Deploy en Desarrollo') {
			when {
				expression { ejecutarDeploy }
			}
			steps {
				echo "Iniciando Deploy en Desarrollo"

				// Despliega el c�digo en el servidor de Natural.
				// C:\apache-ant-1.10.11\bin\ant.bat -file deploy.xml -Dnatural.ant.project.rootdir=../.. -lib C:\workspaces\DevOpsNat\NO4Jenkins\deploy build && exit %%ERRORLEVEL%%

				script {
					def Parametros = "-file ${naturalProyecto}/${naturalProyecto}/deploy.xml -Dnatural.ant.project.rootdir=../.. -lib ${libreriasDeploy} build && exit %%ERRORLEVEL%%"
					withAnt(installation: 'Ant Local', jdk: 'Java') {
						if (isUnix()) {
							sh "ant ${Parametros}"
						}
						else {
							bat "ant ${Parametros}"
						}
					}
				}

				echo "Finalizando Deploy en Desarrollo"
			}
		}

		stage('Arrancando monitorizaci�n Adabas (TPAI)') {
			when {
				expression { ejecutarTPAI }
			}
			steps {
				echo "Iniciando arranque monitorizaci�n Adabas (TPAI)"

				httpRequest url: "${urlWebMethods}/ws/giss.ccd.natDevOps.ntdo.tpai.ws:tpaiService/giss_ccd_natDevOps_ntdo_tpai_ws_tpaiService_Port",
					httpMode: 'POST',
					customHeaders: [[maskValue: false, name: 'SOAPAction', value: 'giss_ccd_natDevOps_ne@Rtdo_tpai_ws_tpaiService_Binder_iniciarMonitores']],
					timeout: 200,
					requestBody: '''<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://seg-social.es/ccd/tpai/service">
					   <soapenv:Header/>
					   <soapenv:Body>
					      <ser:iniciarMonitores/>
					   </soapenv:Body>
					</soapenv:Envelope>''',
					consoleLogResponseBody: true,
			//		responseHandle: 'NONE',
					validResponseContent: '<resultado>1</resultado>',
					wrapAsMultipart: false

				echo "Finalizando arranque monitorizaci�n Adabas (TPAI)"
			}
		}

		stage('Pruebas unitarias (Natural Unit Test)') {
			when {
				expression { ejecutarUnitTest }
			}
			steps {
				echo "Iniciando Pruebas unitarias (Natural Unit Test)"

//				bat "C:/Users/99gu2142/git/PoCNatDevOps/GISSPoCNatDevOps/runUnitTest.cmd"
//				bat "GISSPoCNatDevOps/GISSPoCNatDevOps/runUnitTest.cmd"

				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
			}
		}

		stage('Pruebas funcionales (ALM - UFT)') {
			when {
				expression { ejecutarUFT }
			}
			steps {
				echo "Iniciando Pruebas funcionales (ALM - UFT)"

				node ('UFT_AGENT') {

					runFromAlmBuilder almServerName: 'ALMServer',
						almDomain: "${uftDominio}",
						almProject: "${uftProyecto}",
						almUserName: "${uftUsuario}",
						almPassword: "${uftPassword}",
						almRunMode: 'RUN_REMOTE',
						almRunHost: '10.99.104.203',
						almTestSets: "${uftTestSets}",
						almTimeout: '2000',
						almClientID: '',
						almRunResultsMode: '',
						almApiKey: '',
						isSSOEnabled: false	

				}

				echo "Finalizando Pruebas funcionales (ALM - UFT)"
			}
		}

		stage('Parando monitorizaci�n Adabas (TPAI)') {
			when {
				expression { ejecutarTPAI }
			}
			steps {
				echo "Iniciando parada monitorizaci�n Adabas (TPAI)"

				httpRequest url: "${urlWebMethods}/ws/giss.ccd.natDevOps.ntdo.tpai.ws:tpaiService/giss_ccd_natDevOps_ntdo_tpai_ws_tpaiService_Port",
					httpMode: 'POST',
					customHeaders: [[maskValue: false, name: 'SOAPAction', value: 'giss_ccd_natDevOps_ntdo_tpai_ws_tpaiService_Binder_pararMonitores']],
					timeout: 200,
					requestBody: '''<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://seg-social.es/ccd/tpai/service">
					   <soapenv:Header/>
					   <soapenv:Body>
					      <ser:pararMonitores/>
					   </soapenv:Body>
					</soapenv:Envelope>''',
					consoleLogResponseBody: true,
//					responseHandle: 'NONE',
					validResponseContent: '<resultado>1</resultado>',
					wrapAsMultipart: false

				echo "Finalizando parada monitorizaci�n Adabas (TPAI)"
			}
		}

		stage('Obteniendo an�lisis monitorizaci�n Adabas (TPAI)') {
			when {
				expression { ejecutarTPAI }
			}
			steps {
				echo "Iniciando an�lisis monitorizaci�n Adabas (TPAI)"


				echo "Finalizando an�lisis monitorizaci�n Adabas (TPAI)"
			}
		}

	}

//	post {
//	    always {
//    	    emailext attachLog: true,
//    	    	recipientProviders: [developers(), requestor()],
//    	    	to: 'raul-carlos.de-vicente@at.seg-social.es; jesus.gomez3@at.seg-social.es',
//    	    	subject: '[Jenkins] Build realizada.',
//    	    	body: 'Se ha realizado un Build de la aplicaci�n. '
//    	}
//	}
}