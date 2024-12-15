pipeline{
    options { 
        disableConcurrentBuilds() 
    }
    agent any
    stages{
        stage('GetCode'){
            steps{
                git branch:'develop', url:'https://github.com/david-gallego-campos/UNIR_devops_David.git'
            }
        }
        stage('Tests')
        {
            parallel{
                stage('Unit'){
                    agent {
                        label "agent1"
                    }
                    steps{
                        bat '''
                            set PYTHONPATH=%WORKSPACE%
                            pytest --junitxml=result-unit.xml test\\unit
                            stash includes: '*', name: 'Files'
                        '''
                    }
                }
                stage('Rest'){
                    agent {
                        label "agent2"
                    }
                    steps{
                        bat '''
                            unstash name: 'Files'
                            set FLASK_APP=app\\api.py
                            start flask run 
                            start java -jar wiremock-standalone-3.10.0.jar
                            set PYTHONPATH=%WORKSPACE%
                            pytest --junitxml=result-rest.xml test\\rest
                        '''
                    }
                }
            }
        }
        stage('Results'){
            steps{
                junit 'result*.xml'
            }
        }
    }
}
