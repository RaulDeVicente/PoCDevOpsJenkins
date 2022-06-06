// Pipeline de la PoC del Testing en CE de las aplicaciones Natural.

// Constantes, estas variables deberán estar definidas como variables de entorno de Jenkins:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'


// Variables que definen los datos del proyecto/aplicación
def gitRepositorio = 'PoCDevOpsSelenium'
def codigoAplicacion = 'NTDO'
def uftDominio = 'CCD'
def uftProyecto = 'DEVOPS_PC'
def uftUsuario = 'JENKINPC'
def uftPassword = 'JENKINPC01'
def uftTestSets = '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA'''
def uftEjecutor = '10.99.104.203'

//Variable de sistema 'webdriver.edge.driver' con la ruta del .exe encargado de realizar las pruebas
def webdriverMSEdge= "C:\\edgedriver_win64\\msedgedriver.exe"
//Variable de sistema 'webdriver.chrome.driver' con la ruta del .exe encargado de realizar las pruebas
def webdriverChrome= "C:\\chromedriver.exe"

def ChromeChoice= "Chrome WebDriver"
def MSEdgeChoice= "MSEdge WebDriver"


// Variables que se calculan en el Pipe.
// Variable con el Ticket generada por MONADA.
def MONADA_Ticket
// Variable con el Código de Resultado del inicio de las pruebas con MONADA.
def MONADA_respuesta


pipeline {
	parameters {
		string(name: 'RELEASE', defaultValue: '1.1.1.', description: 'Release asociada.')
		booleanParam(name: 'EJECUTAR_MONADA', defaultValue: true, description: 'Define si se deben ejecutar los Stage de MONADA.')
		booleanParam(name: 'EJECUTAR_UFT', defaultValue: true, description: 'Define si se debe ejecutar el Stage de pruebas funcionales con UFT.')
		booleanParam(name: 'EJECUTAR_SELENIUM_TESTNG', defaultValue: true, description: 'Define si se debe ejecutar el Stage de pruebas funcionales con Selenium y TestNG.')
		string(name: 'seleniumHost', defaultValue: '10.199.55.1', description: 'Dirección del Servidor Pros@ en el que se ejecutarán las pruebas de Selenium.')
		string(name: 'seleniumPort', defaultValue: '9081', description: 'Puerto del Servidor Pros@ en el que se ejecutarán las pruebas de Selenium.')
		choice(name: 'WebDriver', choices: ['Chrome WebDriver', 'MSEdge WebDriver'], description: 'Tipo de WebDriver a utilizar en la prueba.')

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

					monAdabasIniciaPrueba aplicacion: "${codigoAplicacion}",
						version: "${RELEASE}",
						estadoPruebas: 'Failure',
						listaPruebas: [[alcance: '1', elemento: 'TESTT', tipoPrueba: 'O', usuario: 'IDUSE306'],
									   [alcance: '1', elemento: 'PRPI', tipoPrueba: 'P', usuario: 'IDUS7143']]

					def monAdaOutput = readJSON file: "${env.WORKSPACE}/monitorizacionAdabas/${env.BUILD_ID}/iniciarPruebaOutput.json"

					MONADA_Ticket = monAdaOutput.respuestaServicio.ticketPrueba
					MONADA_respuesta = monAdaOutput.respuesta
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

//						almTestSets: "Root\\NTDO\\${RELEASE}\\Funcionales_UFT_${RELEASE}_${env.BUILD_ID}",

					runFromAlmBuilder almServerName: 'ALMServer',
						almCredentialsScope: 'JOB',
						almDomain: "${uftDominio}",
						almProject: "${uftProyecto}",
						almUserName: "${uftUsuario}",
						almPassword: "${uftPassword}",
						areParametersEnabled: false,
						specifyParametersModel: [parameterJson: '[]'],
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

		stage('Pruebas funcionales (Selenium y TestNG)') {
			when {
				expression { params.EJECUTAR_SELENIUM_TESTNG }
			}
			steps {
				echo "Iniciando Pruebas funcionales (Selenium y TestNG)"
                checkout([$class: 'GitSCM',
    					branches: [[name: '*/main']],
    					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])
	   
		        withMaven {
					//Ejecución con ChromeDriver   
					script {
						if (WebDriver == ChromeChoice) {
							bat "mvn -e -f GISSPocDevOpsTestSelenium/pom.xml clean install -Dwebdriver.chrome.driver=${webdriverChrome} -Dhost=${seleniumHost} -Dport=${seleniumPort} -Dmaven.test.failure.ignore=true"
						} else {
							bat "mvn -e -f GISSPocDevOpsTestSelenium/pom.xml clean install -Dwebdriver.edge.driver=${webdriverMSEdge} -Dhost=${seleniumHost} -Dport=${seleniumPort} -Dmaven.test.failure.ignore=true"
						}
					}

					//bat "mvn -e -f GISSPocDevOpsTestSelenium/pom.xml clean install -Dwebdriver.chrome.driver=${webdriverChrome} -Dhost=${seleniumHost} -Dport=${seleniumPort} -Dmaven.test.failure.ignore=true"
					//bat "mvn -e -f GISSPocDevOpsTestSelenium/pom.xml clean install -Dwebdriver.edge.driver=${webdriverMSEdge} -Dhost=${seleniumHost} -Dport=${seleniumPort} -Dmaven.test.failure.ignore=true"
                }

				// Publicando resultados de TestNG
				echo "Publicando resultado de TestNG"
				step([$class: 'Publisher', reportFilenamePattern: '**/testng-results.xml'])

				echo "Publicando resultado en ALM"
// Faltaría ver cómo meter todos los datos del fallo en ALM.
// También falta ver cómo resilver el caso del estado Failed.
				commonResultUploadBuilder almDomain: 'CCD',
					almProject: 'DEVOPS_PC',
					almServerName: 'ALMServer',
					almTestFolder: "NTDO\\${RELEASE}",
					almTestSetFolder: "NTDO\\${RELEASE}",
					clientType: '',
					createNewTest: true,
					credentialsId: 'AlmUser',
					fieldMapping: '''testset:

  root: "x:result/suites/suite"
  name: "v:Funcionales_|x:enclosingBlockNames/string|v:_|${RELEASE}|v:_|${BUILD_ID}"
  subtype-id: "v:hp.qc.test-set.external"
test:
  root: "x:cases/case"
  name: "x:testName"
  testing-framework: "TestNG"
  testing-tool: "Selenium"
  ut-class-name: "x:className"
  ut-method-name: "x:testName"
  subtype-id: "v:EXTERNAL-TEST"
run:
  root: "x:."
  duration: "x:duration"
  status: "x:errorDetails"
  detail: "x:errorDetails"
''',
					runStatusMapping: '''status:
  Failed: "!=NULL"
''',
					testingResultFile: '**/junitResult.xml'


				echo "Finalizando Pruebas funcionales (Selenium y TestNG)"
			}
		}

		stage('Parando monitorización Adabas (MONADA)') {
			when {
				expression { params.EJECUTAR_MONADA }
			}
			steps {
				echo "Iniciando parada monitorización Adabas (MONADA) para el Ticket ${MONADA_Ticket}"

				monAdabasFinalizaPrueba ticketPrueba: "${MONADA_Ticket}",
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

				echo "Falta acceder al Despool a por el informe para el Ticket ${MONADA_Ticket}"
// TODO Falta recoger el informe de Despool.

				echo "Finalizando análisis monitorización Adabas (MONADA)"
			}
		}

	}

}