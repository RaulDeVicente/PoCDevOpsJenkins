// Pipeline de la PoC del IC de las aplicaciones Natural.

// Variable con la puntuaci�n obtenida en Kiuwan.
def KiuwanScore

pipeline {
	agent any

	tools {
	   jdk "Java"
	}

	stages {

		stage('CheckOut Natural Git') {
			steps {
				echo "Iniciando CheckOut de Git"

				// Obtiene el c�digo del GitHub repository
				// git branch:'main', url: 'https://github.com/RaulDeVicente/PoCNatDevOps.git'
				checkout([$class: 'GitSCM',
					branches: [[name: '*/main']],
					extensions: [[$class: 'RelativeTargetDirectory',
					relativeTargetDir: 'GISSPoCNatDevOps']],
					userRemoteConfigs: [[url: 'https://github.com/RaulDeVicente/PoCNatDevOps.git']]])

				echo "Finalizando CheckOut de Git"
			}
		}

		stage('Deploy en Desarrollo') {
			steps {
				echo "Iniciando Deploy en Desarrollo"

				// Despliega el c�digo en el servidor de Natural.
				// C:\apache-ant-1.10.11\bin\ant.bat -file deploy.xml -Dnatural.ant.project.rootdir=../.. -lib C:\workspaces\DevOpsNat\NO4Jenkins\deploy build && exit %%ERRORLEVEL%%

				script {
					def Parametros = '-file GISSPoCNatDevOps/GISSPoCNatDevOps/deploy.xml -Dnatural.ant.project.rootdir=../.. -lib C:/workspaces/DevOpsNat/NO4Jenkins/deploy build && exit %%ERRORLEVEL%%'
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

		stage('An�lisis de c�digo (Kiuwan)') {
			steps {
				echo "Iniciando An�lisis de c�digo (Kiuwan)"

				script {

					kiuwan connectionProfileUuid: 'pqvj-J6Ik', applicationName_dm: 'GISSPoCNatDevOps', selectedMode: 'DELIVERY_MODE', sourcePath: 'GISSPoCNatDevOps/GISSPoCNatDevOps', indicateLanguages_dm: true, languages_dm: 'natural', waitForAuditResults_dm: true

					def kiuwanOutput = readJSON file: "${env.WORKSPACE}/kiuwan/output.json"
					KiuwanScore = kiuwanOutput.auditResult.score
				}

				echo "Finalizando An�lisis de c�digo (Kiuwan) con Score: ${KiuwanScore}"
			}
		}

		stage('Arrancando monitorizaci�n Adabas (TPAI)') {
			steps {
				echo "Iniciando arranque monitorizaci�n Adabas (TPAI)"


				echo "Finalizando arranque monitorizaci�n Adabas (TPAI)"
			}
		}

		stage('Pruebas unitarias (Natural Unit Test)') {
			steps {
				echo "Iniciando Pruebas unitarias (Natural Unit Test)"

//				bat "C:/Users/99gu2142/git/PoCNatDevOps/GISSPoCNatDevOps/runUnitTest.cmd"
//				bat "GISSPoCNatDevOps/GISSPoCNatDevOps/runUnitTest.cmd"

				echo "Finalizando Pruebas unitarias (Natural Unit Test)"
			}
		}

		stage('Pruebas funcionales (ALM - UFT)') {
			steps {
				echo "Iniciando Pruebas funcionales (ALM - UFT)"


				echo "Finalizando Pruebas funcionales (ALM - UFT)"
			}
		}

		stage('Parando monitorizaci�n Adabas (TPAI)') {
			steps {
				echo "Iniciando parada monitorizaci�n Adabas (TPAI)"


				echo "Finalizando parada monitorizaci�n Adabas (TPAI)"
			}
		}

		stage('Obteniendo an�lisis monitorizaci�n Adabas (TPAI)') {
			steps {
				echo "Iniciando an�lisis monitorizaci�n Adabas (TPAI)"


				echo "Finalizando an�lisis monitorizaci�n Adabas (TPAI)"
			}
		}

	}

	post {
	    always {
    	    emailext attachLog: true,
    	    	recipientProviders: [developers(), requestor()],
    	    	to: 'raul-carlos.de-vicente@at.seg-social.es',
    	    	subject: '[Jenkins] Build realizada.',
    	    	body: 'Ejemplo de cuerpo del mensaje.'
    	}
	}
}