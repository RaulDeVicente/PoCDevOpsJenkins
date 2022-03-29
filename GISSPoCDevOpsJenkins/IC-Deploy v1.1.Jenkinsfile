// Pipeline de la PoC para la instalaci�n en IC de las aplicaciones Natural en su versi�n 1.1.

// Constantes, estas variables deber�n estar definidas como variables de entorno de Jenkins:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'
// Variable con la ubicaci�n de las librer�as necesarias para realizar el deploy de Natural.
def libreriasDeploy = 'C:/workspaces/DevOpsNat/NO4Jenkins/deploy914'
// Variable con la ubicaci�n de las librer�as necesarias para realizar el Unit Test de Natural.
def libreriasUnitTest = 'C:/workspaces/DevOpsNat/NO4Jenkins/unitTest'


// Variables que definen los datos del proyecto/aplicaci�n
def gitRepositorio = 'PoCNatDevOps'
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'
def release = "IC_1.1_${env.BUILD_ID}"

// Variables que se calculan en el Pipe.
// Variable con la puntuaci�n obtenida en Kiuwan.
def KiuwanScore
// Variable con el C�digo de Resultado de la Entrega a la Promoci�n Natural.
def entregaRetorno
// Variable con el n�mero de m�dulos Entregados a la Promoci�n Natural.
def entregaModulosProcesados
// Variable con el C�digo de Resultado de la Instalaci�n en CE.
def instalarRetorno


pipeline {
	parameters {
		booleanParam(name: 'EJECUTAR_CHECKOUT', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Checkout de Git.')
		booleanParam(name: 'EJECUTAR_KIUWAN', defaultValue: true, description: 'Define si se debe ejecutar el Stage de An�lisis de c�digo est�tico con Kiuwan.')
		booleanParam(name: 'EJECUTAR_DEPLOYIC', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Despliegue en IC.')
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
				// Obtiene el c�digo del GitHub repository con el Plugin de GIT
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
				expression { params.EJECUTAR_KIUWAN }
			}
			steps {
				echo "Iniciando An�lisis de c�digo (Kiuwan)"

				script {

					kiuwan connectionProfileUuid: 'pqvj-J6Ik',
						applicationName_dm: "${codigoAplicacion}",
						label_dm: "${release}",
						selectedMode: 'DELIVERY_MODE',
						analysisScope_dm: 'PARTIAL_DELIVERY',
						changeRequestStatus_dm: 'INPROGRESS',
						sourcePath: "${naturalProyecto}/${naturalProyecto}/Natural-Libraries",
						indicateLanguages_dm: true,
						languages_dm: 'natural',
						timeout_dm: 30

//					def kiuwanOutput = readJSON file: "${env.WORKSPACE}/kiuwan/output.json"
//					KiuwanScore = kiuwanOutput.auditResult.score
				}

//				echo "Finalizando An�lisis de c�digo (Kiuwan) con Score: ${KiuwanScore}"
				echo "Finalizando An�lisis de c�digo (Kiuwan)"
			}
		}

		stage('Despliegue en CE') {
			when {
				expression { params.EJECUTAR_DEPLOYIC }
			}
			steps {
				echo "Iniciando Despliegue en IC"

				// Despliega el c�digo en el servidor de Natural.
				script {
// TODO Ver c�mo parametrizar el servidor/fuser de entrega para el Ant de despliegue.
					def Parametros = "-file ${naturalProyecto}/${naturalProyecto}/deployICv1.1.xml -Dnatural.ant.project.rootdir=../.. -lib ${libreriasDeploy} build && exit %%ERRORLEVEL%%"
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
					echo "Se ejecuta la Entrega de Release a Promoci�n Natural"
					entregarRelease aplicacion: "${codigoAplicacion}",
						version: "${release}",
						proceso: 'IC',
						rutaFichero: "${env.WORKSPACE}/${naturalProyecto}/${naturalProyecto}",
						estadoRetorno: 'Failure'

					def entregaOutput = readJSON file: "${env.WORKSPACE}/promocionNatural/entregarReleaseOutput_${env.BUILD_ID}.json"

					entregaRetorno = entregaOutput.respuesta
					entregaModulosProcesados = entregaOutput.modulosProcesados

					echo "Se ha ejecutado la Entrega de Release a Promoci�n Natural con respuesta: ${entregaRetorno} y un n�mero de m�dulos entregados: ${entregaModulosProcesados}"

				}

				// Ejecuta la instalaci�n de la release en el entorno de IC
				script {
					echo "Se ejecuta la instalaci�n de la release en el entorno de IC"
					desplegarRelease aplicacion: "${codigoAplicacion}",
						version: "${release}",
						entornoDestino: 'IC',
						estadoRetorno: 'Failure',
						intervaloPooling: "20",
						timeoutPooling: "1200"

					def instalarOutput = readJSON file: "${env.WORKSPACE}/promocionNatural/desplegarReleaseOutput_${env.BUILD_ID}.json"

					instalarRetorno = instalarOutput.respuesta

					echo "Se ha ejecutado la instalaci�n de la release en el entorno de IC con respuesta: ${instalarRetorno}"
				}

				echo "Finalizando Despliegue en IC"
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