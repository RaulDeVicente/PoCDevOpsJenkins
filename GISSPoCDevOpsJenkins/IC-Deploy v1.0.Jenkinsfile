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
						applicationName_dm: "${naturalProyecto}",
						label_dm: "#${release}",
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