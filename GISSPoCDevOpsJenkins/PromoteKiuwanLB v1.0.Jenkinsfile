// Pipeline de la promoci�n a L�nea Base de una release en Kiuwan en su versi�n 1.0.

// Constantes, estas variables deber�n estar definidas como variables de entorno de Jenkins:
// Variables que definen los datos del proyecto/aplicaci�n
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'

pipeline {
	parameters {
		string(name: 'DELIVERY_LABEL', defaultValue: '1.1.1.20', description: 'Etiqueta de la release que debe promocionarse a L�nea Base.')
		string(name: 'BASELINE_LABEL', defaultValue: 'LB-1.1.1.20.1', description: 'Etiqueta de la L�nea Base.')
	}

	agent any

	tools {
	   jdk "Java11"
	}

	stages {

		stage('Promocionar a L�nea Base en Kiuwan') {
			steps {
				echo "Iniciando promoci�n a L�nea Base en Kiuwan de la app. ${codigoAplicacion} y release ${DELIVERY_LABEL} a ${BASELINE_LABEL}"

				script {

					kiuwan connectionProfileUuid: 'pqvj-J6Ik',
						selectedMode: 'EXPERT_MODE',
						sourcePath: "${naturalProyecto}/${naturalProyecto}/Natural-Libraries",
						commandArgs_em: '--promote-to-baseline -n "NTDO" -cr "" -l "${DELIVERY_LABEL}" -pbl "${BASELINE_LABEL}"'

// El sourcePath es obligatorio pasarlo aunque no hace nada con �l. En este caso ni existe.
// No consigo que funcione con el c�digo de la aplicaci�n como una variable.
//						commandArgs_em: '--promote-to-baseline -n "${codigoAplicacion}" -cr "" -l "${DELIVERY_LABEL}" -pbl "${BASELINE_LABEL}"'

				}

				echo "Finalizando promoci�n a L�nea Base en Kiuwan de la app. ${codigoAplicacion} y release ${DELIVERY_LABEL} a ${BASELINE_LABEL}"
			}
		}

	}

}