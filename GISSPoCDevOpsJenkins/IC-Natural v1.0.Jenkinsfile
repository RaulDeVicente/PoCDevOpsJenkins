// Pipeline de la PoC del IC de las aplicaciones Natural en su versi�n 1.0.

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


// Variables que se calculan en el Pipe.
// Variable con la puntuaci�n obtenida en Kiuwan.
def KiuwanScore


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

				// Obtiene el c�digo del GitHub repository con el Plugin de GIT
				checkout([$class: 'GitSCM',
					branches: [[name: '*/main']],
					extensions: [[$class: 'RelativeTargetDirectory',
					relativeTargetDir: "${naturalProyecto}"]],
					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])

				// Obtiene el c�digo del GitHub repository con el Ant de NaturalOne
//				script {
//					def Parametros = "-file ${naturalProyecto}/${naturalProyecto}/deploy.xml -Dnatural.ant.project.rootdir=../.. -lib ${libreriasDeploy} update && exit %%ERRORLEVEL%%"
//					withAnt(installation: 'Ant Local', jdk: 'Java11') {
//						if (isUnix()) {
//							sh "ant ${Parametros}"
//						}
//						else {
//							bat "ant ${Parametros}"
//						}
//					}
//				}

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

// TODO Cambiar el scope a solo cambios
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
					def Parametros = "-lib ${libreriasUnitTest} -buildfile ${naturalProyecto}/${naturalProyecto}/unitTest914.xml -listener com.softwareag.natural.unittest.ant.framework.NaturalTestingJunitLogger -Dnatural.ant.project.rootdir=../.."
					withAnt(installation: 'Ant Local', jdk: 'Java11') {
                 		if (isUnix()) {
							sh "ant ${Parametros}"
						}
						else {
							bat "ant ${Parametros}"
						}
					}
				}
				echo "${env.WORKSPACE}/logUnitTest.xml"
				junit "${env.WORKSPACE}/logUnitTest.xml"

				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
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