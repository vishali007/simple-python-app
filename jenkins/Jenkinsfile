pipeline {
    agent none
    stages {
        stage('Initialize'){
            def dockerHome = tool 'myDocker'
            env.PATH = "${dockerHome}/bin:${env.PATH}"
        }
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
    }
}