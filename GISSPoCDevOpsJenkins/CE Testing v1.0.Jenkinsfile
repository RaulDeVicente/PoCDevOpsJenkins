// Pipeline de la PoC del Testing en CE de las aplicaciones Natural.

// Constantes, estas variables deberán estar definidas como variables de entorno de Jenkins:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'


// Variables que definen los datos del proyecto/aplicación
def gitRepositorio = 'PoCNatDevOps'
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'
def uftDominio = 'CCD'
def uftProyecto = 'DEVOPS_PC'
def uftUsuario = 'JENKINPC'
def uftPassword = 'JENKINPC01'
def uftTestSets = '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA'''
def uftEjecutor = '10.99.104.203'

//Variable que define el host en el que se realizarán las pruebas con Selenium
def host= "10.199.54.174"
def port= "9080"
//http://10.199.54.174:9080/ProsaPortal7/index.jsp


// Variables que se calculan en el Pipe.
// Variable con la puntuación obtenida en Kiuwan.
def KiuwanScore
// Variable con el Ticket generada por MONADA.
def MONADA_Ticket
// Variable con el Código de Resultado de la Entrega a la Promoción Natural.
def entregaRetorno
// Variable con el número de módulos Entregados a la Promoción Natural.
def entregaModulosProcesados
// Variable con el Código de Resultado de la Instalación en CE.
def instalarRetorno
// Variable con el Código de Resultado del inicio de las pruebas con MONADA.
def MONADA_respuesta


pipeline {
	parameters {
		string(name: 'RELEASE', defaultValue: '1.1.1.', description: 'Release asociada.')
		booleanParam(name: 'EJECUTAR_MONADA', defaultValue: true, description: 'Define si se deben ejecutar los Stage de MONADA.')
		booleanParam(name: 'EJECUTAR_UFT', defaultValue: true, description: 'Define si se debe ejecutar el Stage de pruebas funcionales con UFT.')
		booleanParam(name: 'EJECUTAR_SELENIUM_JUNIT', defaultValue: true, description: 'Define si se debe ejecutar el Stage de pruebas funcionales con Selenium y JUnit.')
		booleanParam(name: 'EJECUTAR_SELENIUM_TESTNG', defaultValue: true, description: 'Define si se debe ejecutar el Stage de pruebas funcionales con Selenium y TestNG.')
	}

	agent any

	tools {
	   jdk "Java11"
	}

	stages {

		stage('Arrancando monitorización Adabas (MONADA)') {
			when {
				expression { params.EJECUTAR_MONADA }
			}
			steps {
				echo "Iniciando arranque monitorización Adabas (MONADA)"

				script {

					tpaiIniciaPrueba aplicacion: "${codigoAplicacion}",
						version: "${RELEASE}",
						estadoPruebas: 'Failure',
						listaPruebas: [[alcance: '1', elemento: 'TESTT', tipoPrueba: 'O', usuario: 'IDUSE306'],
									   [alcance: '1', elemento: 'PRPI', tipoPrueba: 'P', usuario: 'IDUSE343']]

					def tpaiOutput = readJSON file: "${env.WORKSPACE}/tpai/iniciarPruebaOutput_${env.BUILD_ID}.json"

					MONADA_Ticket = tpaiOutput.respuestaServicio.ticketPrueba
					MONADA_respuesta = MONADAOutput.respuesta
				}

				echo "Finalizando arranque monitorización Adabas (MONADA) con respuesta ${MONADA_respuesta} y con Ticket ${MONADA_Ticket}"
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
						almCredentialsScope: 'JOB',
						almDomain: "${uftDominio}",
						almProject: "${uftProyecto}",
						almUserName: "${uftUsuario}",
						almPassword: "${uftPassword}",
						almRunMode: 'RUN_REMOTE',
						almRunHost: "${uftEjecutor}",
						almTestSets: "${uftTestSets}",
						almTimeout: '3000',
						almClientID: '',
						almRunResultsMode: '',
						almApiKey: '',
						isSSOEnabled: false	

				}

				echo "Finalizando Pruebas funcionales (ALM - UFT)"
			}
		}

		stage('Pruebas funcionales (Selenium y JUnit)') {
			when {
				expression { params.EJECUTAR_SELENIUM_JUNIT }
			}
			steps {
				echo "Iniciando Pruebas funcionales (Selenium y JUnit)"
                checkout([$class: 'GitSCM',
    					branches: [[name: '*/master']],
    					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])
	   
		        withMaven {
					//Ejecución con ChromeDriver   
					//bat "mvn -e -f GISSPocDevOpsTestSeleniumJUnit/pom.xml clean install -Dwebdriver.chrome.driver=${webdriverChrome} -Dhost=${host} -Dport=${port} -Dmaven.test.failure.ignore=true"
					bat "mvn -e -f GISSPocDevOpsTestSeleniumJUnit/pom.xml clean install -Dwebdriver.edge.driver=${webdriverMSEdge} -Dhost=${host} -Dport=${port} -Dmaven.test.failure.ignore=true"
                }

//				junit 'logUnitTest.xml'

				echo "Finalizando Pruebas funcionales (Selenium y JUnit)"
			}
		}


		stage('Pruebas funcionales (Selenium y TestNG)') {
			when {
				expression { params.EJECUTAR_SELENIUM_TESTNG }
			}
			steps {
				echo "Iniciando Pruebas funcionales (Selenium y TestNG)"
                checkout([$class: 'GitSCM',
    					branches: [[name: '*/master']],
    					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])
	   
		        withMaven {
					//Ejecución con ChromeDriver   
					//bat "mvn -e -f GISSPocDevOpsTestSelenium/pom.xml clean install -Dwebdriver.chrome.driver=${webdriverChrome} -Dhost=${host} -Dport=${port} -Dmaven.test.failure.ignore=true"
					bat "mvn -e -f GISSPocDevOpsTestSelenium/pom.xml clean install -Dwebdriver.edge.driver=${webdriverMSEdge} -Dhost=${host} -Dport=${port} -Dmaven.test.failure.ignore=true"
                }

				step([$class: 'Publisher', reportFilenamePattern: '**/testng-results.xml'])

//**/testng-results.xml

				echo "Finalizando Pruebas funcionales (Selenium y TestNG)"
			}
		}

		stage('Parando monitorización Adabas (MONADA)') {
			when {
				expression { params.EJECUTAR_MONADA }
			}
			steps {
				echo "Iniciando parada monitorización Adabas (MONADA) para el Ticket ${MONADA_Ticket}"

				tpaiFinalizaPrueba ticketPrueba: "${MONADA_Ticket}",
					estadoRetorno: 'Unstable',
					intervaloPooling: "20",
					timeoutPooling: "1200"

				echo "Finalizando parada monitorización Adabas (MONADA) para el Ticket ${MONADA_Ticket}"
			}
		}

		stage('Obteniendo análisis monitorización Adabas (MONADA)') {
			when {
				expression { params.EJECUTAR_MONADA }
			}
			steps {
				echo "Iniciando análisis monitorización Adabas (MONADA) para el Ticket ${MONADA_Ticket}"


				echo "Finalizando análisis monitorización Adabas (MONADA)"
			}
		}

	}

}