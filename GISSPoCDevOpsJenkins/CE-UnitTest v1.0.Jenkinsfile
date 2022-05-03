// Pipeline de la PoC para la ejecución de pruebas unitarias en CE de las aplicaciones Natural en su versión 1.0.

// Constantes, estas variables deberán estar definidas como variables de entorno de Jenkins:
// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'
// Variable con la ubicación de las librerías necesarias para realizar el Unit Test de Natural.
def libreriasUnitTest = 'C:/workspaces/DevOpsNat/NO4Jenkins/unitTest'


// Variables que definen los datos del proyecto/aplicación
def gitRepositorio = 'PoCNatDevOps'
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'

// Variables para las pruebas unitarias
def unitTest_EX_BRK = 'ETB038.99g.giss.ss:10100'
def unitTest_EX_SRV = 'RPC/NTDEVOPS/CALLNAT'
def unitTest_EX_RPCUSR = ''
def unitTest_EX_RPCPWD = ''
def unitTest_EX_EXXUSR = ''
def unitTest_EX_EXXPWD = ''


pipeline {
	parameters {
		string(name: 'RELEASE', defaultValue: '1.1.1.', description: 'Release asociada.')
		booleanParam(name: 'EJECUTAR_UNIT_TEST', defaultValue: true, description: 'Define si se debe ejecutar el Stage de pruebas unitarias con Unit Test.')
	}

	agent any

	tools {
	   jdk "Java11"
	}

	stages {

		stage('Pruebas unitarias (Natural Unit Test)') {
			when {
				expression { params.EJECUTAR_UNIT_TEST }
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
  name: "v:Unitarias_|x:enclosingBlockNames/string|v:_|${RELEASE}|v:_|${BUILD_ID}"
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
  Passed: "==0"
''',
					testingResultFile: '**/junitResult.xml'

//  0: "Passed"
//  1: "Failed"

				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
			}
		}

	}

}