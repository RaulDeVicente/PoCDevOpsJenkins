// Pipeline de la promoci�n a L�nea Base de una release en Kiuwan en su versi�n 1.0.

// Constantes, estas variables deber�n estar definidas como variables de entorno de Jenkins:
// Variables que definen los datos del proyecto/aplicaci�n
def codigoAplicacion = 'NTDO'

pipeline {
	parameters {
		string(name: 'DELIVERY_LABEL', defaultValue: 'PruebaFerCompleta', description: 'Etiqueta de la release que debe promocionarse a L�nea Base.')
		string(name: 'BASELINE_LABEL', defaultValue: 'PruebaLB-Raul', description: 'Etiqueta de la L�nea Base.')
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
						commandArgs_em: '--promote-to-baseline -n "${codigoAplicacion}" -cr CR-9999 -l "${DELIVERY_LABEL}" -pbl "${BASELINE_LABEL}"'

				}

				echo "Finalizando promoci�n a L�nea Base en Kiuwan de la app. ${codigoAplicacion} y release ${DELIVERY_LABEL} a ${BASELINE_LABEL}"
			}
		}

	}

}