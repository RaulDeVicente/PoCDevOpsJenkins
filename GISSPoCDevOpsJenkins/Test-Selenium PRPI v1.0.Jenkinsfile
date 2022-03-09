// Variable con la URL de acceso a Git.
def urlGit = 'https://github.com/RaulDeVicente'

// Variables que definen los datos del proyecto/aplicación
def gitRepositorio = 'PoCDevOpsSelenium'

// Variable que define el nombre del proyecto
def proyecto ="GISSPocDevOpsTestSelenium"

//Variable de sistema 'webdriver.chrome.driver' con la ruta del .exe encargado de realizar las pruebas
def webdriverChrome= "C:\\chromedriver.exe"

//Variable de sistema 'webdriver.edge.driver' con la ruta del .exe encargado de realizar las pruebas
def webdriverMSEdge= "C:\\edgedriver_win64\\msedgedriver.exe"

//Variable que define el host en el que se realizarán las pruebas
def host= "10.199.52.106"

//Variable que define el puerto en el que se realizarán las pruebas
def port= "9080"

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
		         //Ejecución con ChromeDriver   
//                 bat "mvn -e -f ${proyecto}/pom.xml clean install -Dwebdriver.chrome.driver=${webdriverChrome} -Dhost=${host} -Dport=${port} -Dmaven.test.failure.ignore=true"
                	bat "mvn -e -f ${proyecto}/pom.xml clean install -Dwebdriver.edge.driver=${webdriverMSEdge} -Dhost=${host} -Dport=${port} -Dmaven.test.failure.ignore=true"
    
		        }

    		    echo "Finalizada prueba del servicio de PRPI con Selenium"
		   
            }
		}
	}
}