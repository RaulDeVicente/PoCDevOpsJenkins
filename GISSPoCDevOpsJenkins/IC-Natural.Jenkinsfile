// Pipeline de la PoC del IC de las aplicaciones Natural.

// Constantes, estas variables deberán estar definidas como variables de entorno de Jenkins:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'
// Variable con la ubicación de las librerías necesarias para realizar el deploy de Natural.
def libreriasDeploy = 'C:/workspaces/DevOpsNat/NO4Jenkins/deploy'
// Variable con la URL de webMethods.
def urlWebMethods = 'http://g99dnsa824-ld.portal.ss:15555'


// Variables que definen los datos del proyecto/aplicación
def gitRepositorio = 'PoCNatDevOps'
def naturalProyecto = 'GISSPoCNatDevOps'
def uftDominio = 'CCD'
def uftProyecto = 'DEVOPS_PC'
def uftUsuario = 'JENKINPC'
def uftPassword = 'JENKINPC01'
def uftTestSets = '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA'''
def uftEjecutor = '10.99.104.203'


// Variables que se calculan en el Pipe.
// Variable con la puntuación obtenida en Kiuwan.
def KiuwanScore
// Variable con el Ticket generada por TPAI.
def TPAI_Ticket


pipeline {
	parameters {
		booleanParam(name: 'EJECUTAR_CHECKOUT', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Checkout de Git.')
		booleanParam(name: 'EJECUTAR_KIUWAN', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Análisis de código estático con Kiuwan.')
		booleanParam(name: 'EJECUTAR_DEPLOY', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Deploy en desarrollo.')
		booleanParam(name: 'EJECUTAR_TPAI', defaultValue: false, description: 'Define si se deben ejecutar los Stage de TPAI.')
		booleanParam(name: 'EJECUTAR_UNIT_TEST', defaultValue: false, description: 'Define si se debe ejecutar el Stage de pruebas unitarias con Unit Test.')
		booleanParam(name: 'EJECUTAR_UFT', defaultValue: false, description: 'Define si se debe ejecutar el Stage de pruebas funcionales con UFT.')
	}

	agent any

	tools {
	   jdk "Java"
	}

	stages {

		stage('CheckOut Natural Git') {
			when {
				expression { params.EJECUTAR_CHECKOUT }
			}
			steps {
				echo "Iniciando CheckOut de Git"

				// Obtiene el código del GitHub repository
				// git branch:'main', url: 'https://github.com/RaulDeVicente/PoCNatDevOps.git'
				checkout([$class: 'GitSCM',
					branches: [[name: '*/main']],
					extensions: [[$class: 'RelativeTargetDirectory',
					relativeTargetDir: "${naturalProyecto}"]],
					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])

				echo "Finalizando CheckOut de Git"
			}
		}

		stage('Análisis de código (Kiuwan)') {
			when {
				expression { params.EJECUTAR_KIUWAN }
			}
			steps {
				echo "Iniciando Análisis de código (Kiuwan)"

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

				echo "Finalizando Análisis de código (Kiuwan) con Score: ${KiuwanScore}"
			}
		}

		stage('Deploy en Desarrollo') {
			when {
				expression { params.EJECUTAR_DEPLOY }
			}
			steps {
				echo "Iniciando Deploy en Desarrollo"

				// Despliega el código en el servidor de Natural.
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

		stage('Arrancando monitorización Adabas (TPAI)') {
			when {
				expression { params.EJECUTAR_TPAI }
			}
			steps {
				echo "Iniciando arranque monitorización Adabas (TPAI)"

				tpaiIniciaPrueba aplicacion: 'GISSPOCNATDEVOPS',
					version: '1.1.1.1',
					rutaFichero: 'C:\\workspaces\\DevOpsNat\\Jenkins\\.jenkins\\workspace\\DevOps Natural\\IC de Natural\\GISSPoCNatDevOps\\GISSPoCNatDevOps',
					patronFichero: 'history_deploy_GISSPoCNatDevOps',
					repPruebas: [[alcance: '1', elemento: 'TESTT', tipoPrueba: 'O', usuario: 'IDUSE306'],
								[alcance: '1', elemento: 'PRPI', tipoPrueba: 'P', usuario: 'IDUSE343']]

				def tpaiOutput = readJSON file: "${env.WORKSPACE}/tpai/output_${env.BUILD_ID}.json"
				TPAI_Ticket = tpaiOutput.ticketPrueba

				echo "Finalizando arranque monitorización Adabas (TPAI) con el Ticket ${TPAI_Ticket}"
			}
		}

		stage('Pruebas unitarias (Natural Unit Test)') {
			when {
				expression { params.EJECUTAR_UNIT_TEST }
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
				expression { params.EJECUTAR_UFT }
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
						almRunHost: "${uftEjecutor}",
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

		stage('Parando monitorización Adabas (TPAI)') {
			when {
				expression { params.EJECUTAR_TPAI }
			}
			steps {
				echo "Iniciando parada monitorización Adabas (TPAI) para el Ticket ${TPAI_Ticket}"

				tpaiFinalizaPrueba "${TPAI_Ticket}"

				echo "Finalizando parada monitorización Adabas (TPAI)"
			}
		}

		stage('Obteniendo análisis monitorización Adabas (TPAI)') {
			when {
				expression { params.EJECUTAR_TPAI }
			}
			steps {
				echo "Iniciando análisis monitorización Adabas (TPAI) para el Ticket ${TPAI_Ticket}"


				echo "Finalizando análisis monitorización Adabas (TPAI)"
			}
		}

	}

//	post {
//	    always {
//    	    emailext attachLog: true,
//    	    	recipientProviders: [developers(), requestor()],
//    	    	to: 'raul-carlos.de-vicente@at.seg-social.es; jesus.gomez3@at.seg-social.es',
//    	    	subject: '[Jenkins] Build realizada.',
//    	    	body: 'Se ha realizado un Build de la aplicación. '
//    	}
//	}
}