// Pipeline de la PoC para la instalaci�n en IC de las aplicaciones Natural en su versi�n 1.0.

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
def release = "IC_1.0_${env.BUILD_ID}"

// Variables para las pruebas unitarias
def unitTest_EX_BRK = 'ETB038.99g.giss.ss:10100'
def unitTest_EX_SRV = 'RPC/NTSILTGA/CALLNAT'
def unitTest_EX_USR = 'SGU2142'
def unitTest_EX_PWD = 'zr7HrKJhjZ2sx4hmrm12Tg'


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

//						analysisScope_dm: 'PARTIAL_DELIVERY',

					kiuwan connectionProfileUuid: 'pqvj-J6Ik',
						applicationName_dm: "${codigoAplicacion}",
						label_dm: "${release}",
						selectedMode: 'DELIVERY_MODE',
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

		stage('Despliegue en IC') {
			when {
				expression { params.EJECUTAR_DEPLOYIC }
			}
			steps {
				echo "Iniciando Despliegue en IC"

				// Despliega el c�digo en el servidor de Natural.
				script {
					def Parametros = "-buildfile ${naturalProyecto}/${naturalProyecto}/deployICv1.0.xml -Dnatural.ant.project.rootdir=../.. -lib ${libreriasDeploy} build && exit %%ERRORLEVEL%%"
					withAnt(installation: 'Ant Local', jdk: 'Java11') {
						if (isUnix()) {
							sh "ant ${Parametros}"
						}
						else {
							bat "ant ${Parametros}"
						}
					}
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
					def Parametros = "-lib ${libreriasUnitTest} -buildfile ${naturalProyecto}/${naturalProyecto}/unitTest914.xml -listener com.softwareag.natural.unittest.ant.framework.NaturalTestingJunitLogger -Dnatural.ant.project.rootdir=../.. -Dnatural.testing.ant.brokerid=${unitTest_EX_BRK} -Dnatural.testing.ant.srvaddr=${unitTest_EX_SRV} -Dnatural.testing.ant.exxuid=${unitTest_EX_USR} -Dnatural.testing.ant.exxpwd=${unitTest_EX_PWD}"
					withAnt(installation: 'Ant Local', jdk: 'Java11') {
                 		if (isUnix()) {
							sh "ant ${Parametros}"
						}
						else {
							bat "ant ${Parametros}"
						}
					}
				}

				script {
					if (fileExists('logUnitTest.xml')) {
						echo 'Existe el fichero logUnitTest.xml'
//						bat "copy \"${env.WORKSPACE}\\logUnitTest.xml\" \"C:\\workspaces\\DevOpsNat\\Jenkins\\.jenkins\\workspace\\DevOps Natural\\PruebaALM\""
					} else {
						echo 'No existe el fichero logUnitTest.xml'
					}
				}

				echo "Ejecutando plugin de JUnit"
				junit allowEmptyResults: true,
					testResults: 'logUnitTest.xml',
					healthScaleFactor: 1.0,
					keepLongStdio: true

				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
			}
		}

	}

}