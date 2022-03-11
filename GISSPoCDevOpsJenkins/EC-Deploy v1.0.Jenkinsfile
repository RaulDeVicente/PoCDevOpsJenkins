// Pipeline de la PoC para la instalación en EC de las aplicaciones Natural en su versión 1.0.

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
def release = "1.1.1.${env.BUILD_ID}"

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

// TODO Cambiar el modo a Baseline
					kiuwan connectionProfileUuid: 'pqvj-J6Ik',
						applicationName: "${codigoAplicacion}",
						label: "${release}",
						sourcePath: "${naturalProyecto}/${naturalProyecto}/Natural-Libraries",
						indicateLanguages: true,
						languages: 'natural',
						timeout: 30,
						failureThreshold: 10.0,
						unstableThreshold: 20.0

					def kiuwanOutput = readJSON file: "${env.WORKSPACE}/kiuwan/output.json"
					KiuwanScore = kiuwanOutput.auditResult.score
				}

				echo "Finalizando Análisis de código (Kiuwan) con Score: ${KiuwanScore}"
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
						version: "${release}",
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
						version: "${release}",
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
					def Parametros = "-lib ${libreriasUnitTest} -file ${naturalProyecto}/${naturalProyecto}/unitTest914.xml -listener com.softwareag.natural.unittest.ant.framework.NaturalTestingJunitLogger -Dnatural.ant.project.rootdir=../.."
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
				commonResultUploadBuilder almDomain: 'CCD',
					almProject: 'DEVOPS_PC',
					almServerName: 'ALMServer',
					almTestFolder: '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA''',
					almTestSetFolder: '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA''',
					clientType: '',
					createNewTest: false,
					credentialsId: 'AlmUser',
					fieldMapping: '''testset:
						  root: "x:/result/suites/suite"
						  name: "x:name"
						  udf|duration: "x:duration"
						  udf|list: "x:list"
						  subtype-id: "v:hp.qc.test-set.default"
						  #subtype-id: "v:hp.qc.test-set.default"
						  #subtype-id: "v:hp.qc.test-set.external"
						  #subtype-id: "v:hp.sse.test-set.process"
						test:
						  root: "x:cases/case"
						  name: "x:testName|v:_|x:testId" # Use "|" to create a combined value, e.g.: <testName element value>_<testId element value>
						  subtype-id: "v:MANUAL"
						  #subtype-id: "v:SERVICE-TEST"
						  #subtype-id: "v:SYSTEM-TEST"
						  #subtype-id: "v:VAPI-XP-TEST"
						  #subtype-id: "v:ALT-SCENARIO"
						  #subtype-id: "v:BUSINESS-PROCESS"
						  #subtype-id: "v:FLOW"
						  #subtype-id: "v:LEANFT-TEST"
						  #subtype-id: "v:LR-SCENARIO"
						  #subtype-id: "v:QAINSPECT-TEST"
						run:
						  root: "x:." # "." means the same XML object as defined above in \'test\' entity, all elements defined below are searched from the same root element as \'test\' root
						  #root: "x:/result/suites/suite/cases/case"
						  duration: "x:duration" # This system field is integer, if you want to save a float number, please create a string format UDF
						  status: "x:status"
						  udf|Run On Version: "x:RunOnVersion" # Create an UDF and set its label to "run on version", to trigger the versioning logic
						  udf|Date: "2020-03-30" # Date type field value should be in format "yyyy-mm-dd"
						''',
					runStatusMapping: '''status:
						  True: "Passed" # If status attribute is "True" in report, the run in ALM will be marked as "Passed"
						  False: "Failed" # If status attribute is "False" in report, the run in ALM will be marked as "Failed"
						  1: "Passed" # If status attribute is "1" in report, the run in ALM will be marked as "Passed"
						  0: "Failed" # If status attribute is "0" in report, the run in ALM will be marked as "Failed"
						''',
					testingResultFile: 'logUnitTest.xml'

				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
			}
		}

	}

}