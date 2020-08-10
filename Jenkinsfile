pipeline {
	agent none
	
	stages {
		stage('Build code') {
			agent {
				docker {
					image 'python:3.8-alpine'
				}
			}
			steps {
				sh 'python -m py_compile calculator_new.py'
				stash(name: 'compiled-results', includes: '*.py*')
			}
		}
		
		stage('Test code') {
			agent any
			steps {
				echo "Testing Code"
				sh 'docker pull alpine/flake8'
				sh 'docker run --rm -v $(pwd):/apps alpine/flake8:3.5.0 --output-file -flake8-output.xml calculator_new.py'
			}
			post{
				always {
					junit 'flake8-output.xml'
				}
				failure {
					echo 'Failed'
				}
				success {
					echo 'Done'
				}
			}
		}
		
		stage('Deploy code') {
			agent any
			environment {
				VOLUME = '&(pwd):/src'
				IMAGE = 'cdrx/pyinstaller-windows:python3'
			}
			steps {
				dir(path: env.BUILD_ID) {
					unstash(name: 'compiled_results')
					sh "docker run --rm -v $(VOLUME) $(IMAGE) 'pyinstaller -F calculator_new.py'"
				}
			}
			post {
				success {
					archiveArtifacts "${env.BUILD_ID}/dist/calculator_new.exe"
					sh "docker run --rm -v $(VOLUME) $(IMAGE) 'rm -rf build dist'"
				}
			}
		}
		
