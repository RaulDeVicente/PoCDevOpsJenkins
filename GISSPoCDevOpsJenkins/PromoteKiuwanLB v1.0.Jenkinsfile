// Pipeline de la promoción a Línea Base de una release en Kiuwan en su versión 1.0.

// Constantes, estas variables deberán estar definidas como variables de entorno de Jenkins:
// Variables que definen los datos del proyecto/aplicación
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'

pipeline {
	parameters {
		string(name: 'DELIVERY_LABEL', defaultValue: '1.1.1.20', description: 'Etiqueta de la release que debe promocionarse a Línea Base.')
		string(name: 'BASELINE_LABEL', defaultValue: 'LB-1.1.1.20.1', description: 'Etiqueta de la Línea Base.')
	}

	agent any

	tools {
	   jdk "Java11"
	}

	stages {

		stage('Promocionar a Línea Base en Kiuwan') {
			steps {
				echo "Iniciando promoción a Línea Base en Kiuwan de la app. ${codigoAplicacion} y release ${DELIVERY_LABEL} a ${BASELINE_LABEL}"

				script {

					kiuwan connectionProfileUuid: 'pqvj-J6Ik',
						selectedMode: 'EXPERT_MODE',
						sourcePath: "${naturalProyecto}/${naturalProyecto}/Natural-Libraries",
						commandArgs_em: '--promote-to-baseline -n "NTDO" -cr "" -l "${DELIVERY_LABEL}" -pbl "${BASELINE_LABEL}"'

// El sourcePath es obligatorio pasarlo aunque no hace nada con él. En este caso ni existe.
// No consigo que funcione con el código de la aplicación como una variable.
//						commandArgs_em: '--promote-to-baseline -n "${codigoAplicacion}" -cr "" -l "${DELIVERY_LABEL}" -pbl "${BASELINE_LABEL}"'

				}

				echo "Finalizando promoción a Línea Base en Kiuwan de la app. ${codigoAplicacion} y release ${DELIVERY_LABEL} a ${BASELINE_LABEL}"
			}
		}

	}

}