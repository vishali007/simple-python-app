pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    //This image parameter (of the agent section’s docker parameter) downloads the python:2-alpine
                    //Docker image and runs this image as a separate container. The Python container becomes
                    //the agent that Jenkins uses to run the Build stage of your Pipeline project.
                    image 'python:2-alpine'
                }
            }
            steps {
                //This sh step runs the Python command to compile your application and
                //its calc library into byte code files, which are placed into the sources workspace directory
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                //This stash step saves the Python source code and compiled byte code files from the sources
                //workspace directory for use in later stages.
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }

        stage('Test') {
            agent {
                docker {
                    //This image parameter downloads the qnib:pytest Docker image and runs this image as a
                    //separate container. The pytest container becomes the agent that Jenkins uses to run the Test
                    //stage of your Pipeline project.
                    image 'qnib/pytest'
                }
            }
            steps {
                //This sh step executes pytest’s py.test command on sources/test_calc.py, which runs a set of
                //unit tests (defined in test_calc.py) on the "calc" library’s add2 function.
                //The --junit-xml test-reports/results.xml option makes py.test generate a JUnit XML report,
                //which is saved to test-reports/results.xml
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    //This junit step archives the JUnit XML report (generated by the py.test command above) and
                    //exposes the results through the Jenkins interface.
                    //The post section’s always condition that contains this junit step ensures that the step is
                    //always executed at the completion of the Test stage, regardless of the stage’s outcome.
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            agent any
            //This environment block defines two variables which will be used later in the 'Deliver' stage.
            environment {
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-window:python2'
            }
            steps {
                //This dir step creates a new subdirectory named by the build number.
                //The final program will be created in that directory by pyinstaller.
                //BUILD_ID is one of the pre-defined Jenkins environment variables.
                //This unstash step restores the Python source code and compiled byte
                //code files (with .pyc extension) from the previously saved stash. image]
                //and runs this image as a separate container.
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')

                    //This sh step executes the pyinstaller command (in the PyInstaller container) on your simple Python application.
                    //This bundles your add2vals.py Python application into a single standalone executable file
                    //and outputs this file to the dist workspace directory (within the Jenkins home directory).
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
            post {
                success {
                    //This archiveArtifacts step archives the standalone executable file and exposes this file
                    //through the Jenkins interface.
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}