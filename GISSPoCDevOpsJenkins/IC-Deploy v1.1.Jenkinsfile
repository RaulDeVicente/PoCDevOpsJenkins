// Pipeline de la PoC para la instalaci�n en IC de las aplicaciones Natural en su versi�n 1.1.

// Constantes, estas variables deber�n estar definidas como variables de entorno de Jenkins:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'
// Variable con la ubicaci�n de las librer�as necesarias para realizar el deploy de Natural.
def libreriasDeploy = 'C:/Raul/LibsNatOne/deploy'
// Variable con la ubicaci�n de las librer�as necesarias para realizar el Unit Test de Natural.
def libreriasUnitTest = 'C:/Raul/LibsNatOne/unitTest'


// Variables que definen los datos del proyecto/aplicaci�n
def gitRepositorio = 'PoCNatDevOps'
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'
def release = "IC_1.1_${env.BUILD_ID}"

// Variables para las pruebas unitarias
def unitTest_EX_BRK = 'ETB038.99g.giss.ss:10100'
def unitTest_EX_SRV = 'RPC/NTDEVOPS/CALLNAT'
def unitTest_EX_RPCUSR = ''
def unitTest_EX_RPCPWD = ''
def unitTest_EX_EXXUSR = ''
def unitTest_EX_EXXPWD = ''


// Variables que se calculan en el Pipe.
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
		string(name: 'KIUWAN_ChangeRequest', defaultValue: 'Jira xxxxx', description: 'Nombre del Change Request al que quedar� asociado el an�lisis.')
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

//						analysisScope_dm: 'PARTIAL_DELIVERY',

					kiuwan connectionProfileUuid: 'EZs3-spo7',
						applicationName_dm: "${codigoAplicacion}",
						label_dm: "${release}",
						selectedMode: 'DELIVERY_MODE',
						changeRequest_dm: "${KIUWAN_ChangeRequest}",
						changeRequestStatus_dm: 'INPROGRESS',
						sourcePath: "${naturalProyecto}/${naturalProyecto}/Natural-Libraries",
						indicateLanguages_dm: true,
						languages_dm: 'natural',
						timeout_dm: 30

				}

				echo "Finalizando An�lisis de c�digo (Kiuwan)"
			}
		}

		stage('Despliegue en IC') {
			when {
				expression { params.EJECUTAR_DEPLOYIC }
			}
			steps {
				echo "Iniciando Despliegue en IC"

				// Despliega el c�digo en el servidor de Natural.
				script {
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

					def entregaOutput = readJSON file: "${env.WORKSPACE}/promocionNatural/${env.BUILD_ID}/entregarReleaseOutput.json"

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

					def instalarOutput = readJSON file: "${env.WORKSPACE}/promocionNatural/${env.BUILD_ID}/desplegarReleaseOutput.json"

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
					def Parametros = "-lib ${libreriasUnitTest} -buildfile ${naturalProyecto}/${naturalProyecto}/unitTest914NoLogon.xml -listener com.softwareag.natural.unittest.ant.framework.NaturalTestingJunitLogger -Dnatural.ant.project.rootdir=../.. -Dnatural.testing.ant.brokerid=${unitTest_EX_BRK} -Dnatural.testing.ant.srvaddr=${unitTest_EX_SRV} -Dnatural.testing.ant.rpcuid=${unitTest_EX_RPCUSR} -Dnatural.testing.ant.rpcpwd=${unitTest_EX_RPCPWD} -Dnatural.testing.ant.exxuid=${unitTest_EX_EXXUSR} -Dnatural.testing.ant.exxpwd=${unitTest_EX_EXXPWD}"
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


				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
			}
		}

	}

}