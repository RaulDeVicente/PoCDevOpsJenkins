// Pipeline de la promoción a Línea Base de una release en Kiuwan en su versión 1.0.

// Constantes, estas variables deberán estar definidas como variables de entorno de Jenkins:
// Variables que definen los datos del proyecto/aplicación
def codigoAplicacion = 'NTDO'
def naturalProyecto = 'GISSPoCNatDevOps'

pipeline {
	parameters {
		string(name: 'DELIVERY_LABEL', defaultValue: 'PruebaFerCompleta', description: 'Etiqueta de la release que debe promocionarse a Línea Base.')
		string(name: 'BASELINE_LABEL', defaultValue: 'PruebaLB-Raul', description: 'Etiqueta de la Línea Base.')
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

//kiuwan applicationName_dm: 'PIAC', commandArgs_em: '--', connectionProfileUuid: 'GhBG-3FVp', label_dm: '02.00.00.07', selectedMode: 'DELIVERY_MODE', sourcePath: '/apps/kiuwan/fuentes/GA/PIAC', waitForAuditResults_dm: true
//					kiuwan connectionProfileUuid: 'GhBG-3FVp',
//						selectedMode: 'EXPERT_MODE',
//						sourcePath: '/apps/kiuwan/fuentes/GA/PIAC',
//						commandArgs_em: '--promote-to-baseline -n "PIAC" -cr "" -l "02.00.00.07" -pbl "LB-02.00.00.07"'

					kiuwan connectionProfileUuid: 'pqvj-J6Ik',
						selectedMode: 'EXPERT_MODE',
						sourcePath: "${naturalProyecto}/${naturalProyecto}/Natural-Libraries",
						commandArgs_em: '--promote-to-baseline -n "NTDO" -cr "" -l "${DELIVERY_LABEL}" -pbl "${BASELINE_LABEL}"'

				}

				echo "Finalizando promoción a Línea Base en Kiuwan de la app. ${codigoAplicacion} y release ${DELIVERY_LABEL} a ${BASELINE_LABEL}"
			}
		}

	}

}