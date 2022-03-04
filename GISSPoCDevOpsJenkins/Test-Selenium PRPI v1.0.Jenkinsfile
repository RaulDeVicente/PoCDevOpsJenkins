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
		    
                checkout([$class: 'GitSCM',
    					branches: [[name: '*/main']],
    					userRemoteConfigs: [[url: "${urlGit}/${gitRepositorio}.git"]]])
    		    echo "Finalizando CheckOut de Git y build"
		   
		        withMaven {
                    bat "mvn -f ${proyecto}/pom.xml clean install"
                } // withMaven will discover the generated Maven artifacts, JUnit Surefire & FailSafe reports and FindBugs reports
		   
              }
            //post {
            //    always {
            //      step([$class: 'Publisher', reportFilenamePattern: '**/testng-results.xml', showFailedBuilds:'true' ])
            //    }
		    //}
		}
	}
}