// Pipeline de la PoC para la instalación en IC de las aplicaciones Natural en su versión 1.0.

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
def release = "IC.${env.BUILD_ID}"

pipeline {
	parameters {
		booleanParam(name: 'EJECUTAR_CHECKOUT', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Checkout de Git.')
		booleanParam(name: 'EJECUTAR_KIUWAN', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Análisis de código estático con Kiuwan.')
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
						label_dm: "${release}",
						selectedMode: 'DELIVERY_MODE',
						analysisScope_dm: 'PARTIAL_DELIVERY',
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

		stage('Despliegue en IC') {
			when {
				expression { params.EJECUTAR_DEPLOYIC }
			}
			steps {
				echo "Iniciando Despliegue en IC"

				// Despliega el código en el servidor de Natural.
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

				script {
					if (fileExists('logUnitTest.xml')) {
						echo 'Existe el fichero logUnitTest.xml'
						bat "copy \"${env.WORKSPACE}\\logUnitTest.xml\" \"C:\\workspaces\\DevOpsNat\\Jenkins\\.jenkins\\workspace\\DevOps Natural\\PruebaALM\""
					} else {
						echo 'No existe el fichero logUnitTest.xml'
					}
				}

				echo "Ejecutando plugin de JUnit"
				junit allowEmptyResults: true,
					testResults: 'logUnitTest.xml',
					healthScaleFactor: 1.0,
					keepLongStdio: true


				echo "Publicando resultado en ALM"

//				node ('NodoJava8') {

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
//junitResult.xml
//logUnitTest.xml
//				}

				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
			}
		}

	}

}