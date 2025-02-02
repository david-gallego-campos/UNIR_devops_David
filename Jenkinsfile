pipeline{
    agent any
    stages{
        stage('Get Code'){
            steps{
                git branch:'feature_fix_coverage', url:'https://github.com/david-gallego-campos/UNIR_devops_David.git'
                bat 'dir'
                echo WORKSPACE
            }
        }

        stage('Unit'){
            steps{
                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){ 
                        bat '''
                            set PYTHONPATH=%WORKSPACE%
                            coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                            coverage xml  
                        '''
                        junit 'result*.xml'
                    }
            }
        }
        
        stage('Coverage'){
            steps{
                
                bat '''
                     coverage report  
                '''
                 
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){ 
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,80', lineCoverageTargets: '100,0,85', onlyStable: false
                }
            }
        }
        
        stage('Static'){
            steps{
                bat '''
                    flake8 --exit-zero --format=pylint app>flake8.out
                '''
                
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
            
                }    
            }
        }
        
        stage('Security'){
            steps{
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}:[:{test_id}]:{msg}"
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                }
            }
            
        }
        
        stage('Performance'){
            steps{
                bat '''
                    SET FLASK_APP=app\\api.py
                    start C:\\Users\\User\\AppData\\Local\\Programs\\Python\\Python310\\python.exe -m flask run --host=0.0.0.0 --port:5000
                    ping localhost -n 20
                    C:\\David\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
            
        }
        
        
    }
}
