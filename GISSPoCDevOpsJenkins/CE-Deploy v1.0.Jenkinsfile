// Pipeline de la PoC para la instalación en CE de las aplicaciones Natural en su versión 1.0.

// Constantes, estas variables deberán estar definidas como variables de entorno de Jenkins:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'
// Variable con la ubicación de las librerías necesarias para realizar el deploy de Natural.
def libreriasDeploy = 'C:/workspaces/DevOpsNat/NO4Jenkins/deploy914'
// Variable con la ubicación de las librerías necesarias para realizar el Unit Test de Natural.
def libreriasUnitTest = 'C:/workspaces/DevOpsNat/NO4Jenkins/unitTest'


// Variables que definen los datos del proyecto/aplicación
def gitRepositorio = 'PoCNatDevOps'
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'

// Variables para las pruebas unitarias
def unitTest_EX_BRK = 'ETB038.99g.giss.ss:10100'
def unitTest_EX_SRV = 'RPC/NTSILTGA/CALLNAT'
def unitTest_EX_RPCUSR = 'SGU2142'
def unitTest_EX_RPCPWD = 'zr7HrKJhjZ2sx4hmrm12Tg'
def unitTest_EX_EXXUSR = 'SGU2142'
def unitTest_EX_EXXPWD = 'zr7HrKJhjZ2sx4hmrm12Tg'


// Variables que se calculan en el Pipe.
// Variable con la puntuación obtenida en Kiuwan.
def KiuwanScore
// Variable con el Código de Resultado de la Entrega a la Promoción Natural.
def entregaRetorno
// Variable con el número de módulos Entregados a la Promoción Natural.
def entregaModulosProcesados
// Variable con el Código de Resultado de la Instalación en CE.
def instalarRetorno


pipeline {
	parameters {
		string(name: 'RELEASE', defaultValue: '1.1.1.', description: 'Release asociada.')
		booleanParam(name: 'EJECUTAR_CHECKOUT', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Checkout de Git.')
		booleanParam(name: 'EJECUTAR_KIUWAN', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Análisis de código estático con Kiuwan.')
		booleanParam(name: 'EJECUTAR_DEPLOYCE', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Despliegue en CE.')
		booleanParam(name: 'EJECUTAR_UNIT_TEST', defaultValue: false, description: 'Define si se debe ejecutar el Stage de pruebas unitarias con Unit Test.')
	}

	agent any

	tools {
	   jdk "Java11"
	}

	stages {

		stage('CheckOut Natural Git') {
			when {
				expression { params.EJECUTAR_CHECKOUT }
			}
			steps {
				echo "Iniciando CheckOut de Git"

// TODO Cambiar la rama.
				// Obtiene el código del GitHub repository con el Plugin de GIT
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
						applicationName_dm: "${codigoAplicacion}",
						label_dm: "${RELEASE}",
						selectedMode: 'DELIVERY_MODE',
						sourcePath: "${naturalProyecto}/${naturalProyecto}/Natural-Libraries",
						indicateLanguages_dm: true,
						languages_dm: 'natural',
						timeout_dm: 30,
						waitForAuditResults_dm: true

//					def kiuwanOutput = readJSON file: "${env.WORKSPACE}/kiuwan/output.json"
//					KiuwanScore = kiuwanOutput.auditResult.score
				}

//				echo "Finalizando Análisis de código (Kiuwan) con Score: ${KiuwanScore}"
				echo "Finalizando Análisis de código (Kiuwan)"
			}
		}

		stage('Despliegue en CE') {
			when {
				expression { params.EJECUTAR_DEPLOYCE }
			}
			steps {
				echo "Iniciando Despliegue en CE"

				// Despliega el código en el servidor de Natural.
				script {
// TODO Ver cómo parametrizar el servidor/fuser de entrega para el Ant de despliegue.
					def Parametros = "-file ${naturalProyecto}/${naturalProyecto}/deployECv1.0.xml -Dnatural.ant.project.rootdir=../.. -lib ${libreriasDeploy} build && exit %%ERRORLEVEL%%"
					withAnt(installation: 'Ant Local', jdk: 'Java11') {
						if (isUnix()) {
							sh "ant ${Parametros}"
						}
						else {
							bat "ant ${Parametros}"
						}
					}
				}

				// Ejecuta el servicio de entrega de Release.
				script {
					echo "Se ejecuta la Entrega de Release a Promoción Natural"
					entregarRelease aplicacion: "${codigoAplicacion}",
						version: "${RELEASE}",
						proceso: 'CE',
						rutaFichero: "${env.WORKSPACE}/${naturalProyecto}/${naturalProyecto}",
						estadoRetorno: 'Failure'

					def entregaOutput = readJSON file: "${env.WORKSPACE}/promocionNatural/entregarReleaseOutput_${env.BUILD_ID}.json"

					entregaRetorno = entregaOutput.respuesta
					entregaModulosProcesados = entregaOutput.modulosProcesados

					echo "Se ha ejecutado la Entrega de Release a Promoción Natural con respuesta: ${entregaRetorno} y un número de módulos entregados: ${entregaModulosProcesados}"

				}

				// Ejecuta la instalación de la release en el entorno de CE
				script {
					echo "Se ejecuta la instalación de la release en el entorno de CE"
					desplegarRelease aplicacion: "${codigoAplicacion}",
						version: "${RELEASE}",
						entornoDestino: 'CE',
						estadoRetorno: 'Failure',
						intervaloPooling: "20",
						timeoutPooling: "1200"

					def instalarOutput = readJSON file: "${env.WORKSPACE}/promocionNatural/desplegarReleaseOutput_${env.BUILD_ID}.json"

					instalarRetorno = instalarOutput.respuesta

					echo "Se ha ejecutado la instalación de la release en el entorno de CE con respuesta: ${instalarRetorno}"
				}

				echo "Finalizando Despliegue en CE"
			}
		}

		stage('Pruebas unitarias (Natural Unit Test)') {
			when {
				expression { params.EJECUTAR_UNIT_TEST }
			}
			steps {
				echo "Iniciando Pruebas unitarias (Natural Unit Test)"

				script {
					def Parametros = "-lib ${libreriasUnitTest} -buildfile ${naturalProyecto}/${naturalProyecto}/unitTest914.xml -listener com.softwareag.natural.unittest.ant.framework.NaturalTestingJunitLogger -Dnatural.ant.project.rootdir=../.. -Dnatural.testing.ant.brokerid=${unitTest_EX_BRK} -Dnatural.testing.ant.srvaddr=${unitTest_EX_SRV} -Dnatural.testing.ant.rpcuid=${unitTest_EX_RPCUSR} -Dnatural.testing.ant.rpcpwd=${unitTest_EX_RPCPWD} -Dnatural.testing.ant.exxuid=${unitTest_EX_EXXUSR} -Dnatural.testing.ant.exxpwd=${unitTest_EX_EXXPWD}"
					withAnt(installation: 'Ant Local', jdk: 'Java11') {
						if (isUnix()) {
							sh "ant ${Parametros}"
						}
						else {
							bat "ant ${Parametros}"
						}
					}
				}

				echo "Ejecutando plugin de JUnit"
				junit 'logUnitTest.xml'


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
  name: "x:name|v:_|${RELEASE}"
  subtype-id: "v:hp.qc.test-set.external"
test:
  root: "x:cases/case"
  name: "x:testName"
  subtype-id: "v:EXTERNAL-TEST"
run:
  root: "x:."
  duration: "x:duration"
  status: "x:failedSince"
''',
					runStatusMapping: '''status:
  0: "Passed"
  1: "Failed"
''',
					testingResultFile: '**/junitResult.xml'

// Esta forma no funciona con Java 11, solo con Java 8.
//				uploadResultToALM almServerName: 'ALMServer',
//					credentialsId: 'AlmUser',
//					almDomain: 'CCD',
//					almProject: 'DEVOPS_PC',
//					clientType: '',
//					almTimeout: '600',
//					jenkinsServerUrl: 'http://ntx52desa299.seg-social.ss:8080',
//					almTestFolder: "Prueba\\PruebaFBG\\${env.BUILD_ID}",
//					almTestSetFolder: "Prueba\\PruebaFBG\\${env.BUILD_ID}",
//					testingFramework: 'JUnit',
//					testingResultFile: '**/junitResult.xml',
//					testingTool: 'Natural Unit test'

				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
			}
		}

	}

}