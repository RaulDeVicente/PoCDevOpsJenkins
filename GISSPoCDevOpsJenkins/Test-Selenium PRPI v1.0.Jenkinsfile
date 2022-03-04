// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'

// Variables que definen los datos del proyecto/aplicación
def gitRepositorio = 'PoCDevOpsSelenium'

// Variable que define el nombre del proyecto
def proyecto ="GISSPocDevOpsTestSelenium"


pipeline {

	agent any

	stages {

		stage('Construir Git-Maven') {
		
		    steps {

    		    echo "Iniciando prueba del servicio de PRPI con Selenium"
		    
                checkout([$class: 'GitSCM',
    					branches: [[name: '*/main']],
    					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])
		   
		        withMaven {
                    bat "mvn -f ${proyecto}/pom.xml clean install"
                }

    		    echo "Finalizada prueba del servicio de PRPI con Selenium"
		   
            }
		}
	}
}