// Pipeline de la PoC del IC de las aplicaciones Natural.

// Constantes, estas variables deber�n estar definidas como variables de entorno de Jenkins:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'
// Variable con la ubicaci�n de las librer�as necesarias para realizar el deploy de Natural.
def libreriasDeploy = 'C:/workspaces/DevOpsNat/NO4Jenkins/deploy'
// Variable con la ubicaci�n de las librer�as necesarias para realizar el Unit Test de Natural.
def libreriasUnitTest = 'C:/workspaces/DevOpsNat/NO4Jenkins/unitTest'
// Variable con la URL de webMethods.
def urlWebMethods = 'http://g99dnsa824-ld.portal.ss:15555'


// Variables que definen los datos del proyecto/aplicaci�n
def gitRepositorio = 'PoCNatDevOps'
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'
def version = "1.1.1.${env.BUILD_ID}"
def uftDominio = 'CCD'
def uftProyecto = 'DEVOPS_PC'
def uftUsuario = 'JENKINPC'
def uftPassword = 'JENKINPC01'
def uftTestSets = '''Root\\UFT_2021\\Testing_CI_UFT_2021_DESA'''
def uftEjecutor = '10.99.104.203'


// Variables que se calculan en el Pipe.
// Variable con la puntuaci�n obtenida en Kiuwan.
def KiuwanScore
// Variable con el Ticket generada por TPAI.
def TPAI_Ticket
// Variable con el C�digo de Resultado de la Entrega a la Promoci�n Natural.
def PromNatRetorno


pipeline {
	parameters {
		booleanParam(name: 'EJECUTAR_CHECKOUT', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Checkout de Git.')
		booleanParam(name: 'EJECUTAR_KIUWAN', defaultValue: true, description: 'Define si se debe ejecutar el Stage de An�lisis de c�digo est�tico con Kiuwan.')
		booleanParam(name: 'EJECUTAR_DEPLOY', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Deploy en desarrollo.')
		booleanParam(name: 'EJECUTAR_ENTREGA', defaultValue: true, description: 'Define si se debe ejecutar el Stage de Entregar a Promoci�n Natural.')
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

				// Obtiene el c�digo del GitHub repository con el Plugin de GIT
				checkout([$class: 'GitSCM',
					branches: [[name: '*/main']],
					extensions: [[$class: 'RelativeTargetDirectory',
					relativeTargetDir: "${naturalProyecto}"]],
					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])

				// Obtiene el c�digo del GitHub repository con el Ant de NaturalOne
//				script {
//					def Parametros = "-file ${naturalProyecto}/${naturalProyecto}/deploy.xml -Dnatural.ant.project.rootdir=../.. -lib ${libreriasDeploy} update && exit %%ERRORLEVEL%%"
//					withAnt(installation: 'Ant Local', jdk: 'Java') {
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

		stage('Deploy en Desarrollo') {
			when {
				expression { params.EJECUTAR_DEPLOY }
			}
			steps {
				echo "Iniciando Deploy en Desarrollo"

				// Despliega el c�digo en el servidor de Natural.
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

		stage('Entregar a Promoci�n Natural') {
			when {
				expression { params.EJECUTAR_ENTREGA }
			}
			steps {
				echo "Iniciando Entregar a Promoci�n Natural"

				// Despliega el c�digo en el servidor de Natural.
				script {
					entregarRelease aplicacion: "${codigoAplicacion}",
						version: "${version}",
						rutaFichero: "${env.WORKSPACE}/${naturalProyecto}/${naturalProyecto}",
						patronFichero: 'history_deploy_',
						estadoRetorno: 'Failure',
						selSoloModulosModificados: 'true',
						selTodosTiposModulos: 'true'

					def promNatOutput = readJSON file: "${env.WORKSPACE}/promocionNatural/entregarReleaseOutput_${env.BUILD_ID}.json"

					PromNatRetorno = promNatOutput.codRetorno
				}

				echo "Finalizando Entregar a Promoci�n Natural con resultado ${PromNatRetorno}"
			}
		}




		stage('Arrancando monitorizaci�n Adabas (TPAI)') {
			when {
				expression { params.EJECUTAR_TPAI }
			}
			steps {
				echo "Iniciando arranque monitorizaci�n Adabas (TPAI)"

				script {

					tpaiIniciaPrueba aplicacion: "${codigoAplicacion}",
						version: "${version}",
						rutaFichero: "${env.WORKSPACE}/${naturalProyecto}/${naturalProyecto}",
						patronFichero: 'history_deploy_',
						estadoPruebas: 'Failure',
						selSoloModulosModificados: 'true',
						selTodosTiposModulos: 'false',
						selModulosProgram: true,
						selModulosSubprogram: true,
						selModulosSubroutine: true,
						selModulosFunction: true,
						selModulosClass: true,
						selModulosCopycode: true,
						selModulosDataDefinitionModule: false,
						selModulosDialog: false,
						selModulosGlobalDataArea: false,
						selModulosHelproutine: false,
						selModulosLocalDataArea: false,
						selModulosMap: false,
						selModulosParameterDataArea: false,
						selModulosText: false,
						listaPruebas: [[alcance: '1', elemento: 'TESTT', tipoPrueba: 'O', usuario: 'IDUSE306'],
									   [alcance: '1', elemento: 'PRPI', tipoPrueba: 'P', usuario: 'IDUSE343']]

					def tpaiOutput = readJSON file: "${env.WORKSPACE}/tpai/iniciarPruebaOutput_${env.BUILD_ID}.json"

					TPAI_Ticket = tpaiOutput.ticketPrueba
				}

				echo "Finalizando arranque monitorizaci�n Adabas (TPAI) con el Ticket ${TPAI_Ticket}"
			}
		}

		stage('Pruebas unitarias (Natural Unit Test)') {
			when {
				expression { params.EJECUTAR_UNIT_TEST }
			}
			steps {
				echo "Iniciando Pruebas unitarias (Natural Unit Test)"

				script {
					def Parametros = "-lib ${libreriasUnitTest} -file ${naturalProyecto}/${naturalProyecto}/unitTest914.xml -listener com.softwareag.natural.unittest.ant.framework.NaturalTestingJunitLogger -Dnatural.ant.project.rootdir=."
					withAnt(installation: 'Ant Local', jdk: 'Java') {
						if (isUnix()) {
							sh "ant ${Parametros}"
						}
						else {
							bat "ant ${Parametros}"
						}
					}
				}

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

		stage('Parando monitorizaci�n Adabas (TPAI)') {
			when {
				expression { params.EJECUTAR_TPAI }
			}
			steps {
				echo "Iniciando parada monitorizaci�n Adabas (TPAI) para el Ticket ${TPAI_Ticket}"

				tpaiFinalizaPrueba ticketPrueba: "${TPAI_Ticket}",
					estadoRetorno: 'Unstable'

				echo "Finalizando parada monitorizaci�n Adabas (TPAI)"
			}
		}

		stage('Obteniendo an�lisis monitorizaci�n Adabas (TPAI)') {
			when {
				expression { params.EJECUTAR_TPAI }
			}
			steps {
				echo "Iniciando an�lisis monitorizaci�n Adabas (TPAI) para el Ticket ${TPAI_Ticket}"


				echo "Finalizando an�lisis monitorizaci�n Adabas (TPAI)"
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