pipeline {
	agent any

	stages {

		stage('Pruebecillas') {
			steps {
				echo "Iniciando Pruebecillas"
				
				def Score
				
				script {
					def kiuwanOutput = readJSON file: "C:/workspaces/DevOpsNat/Jenkins/.jenkins/workspace/DevOps Natural/IC de Natural/kiuwan/output.json"
					Score = kiuwanOutput.auditResult.score
				}

				echo "Finalizando Pruebecillas con ${Score}"
			}
		}
	}
}