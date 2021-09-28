pipeline {
    agent any

    options {
        timestamps() // Add timestamps to logging
        timeout(time: 12, unit: 'HOURS') // Abort pipleine
        
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        disableConcurrentBuilds()
    }
    
    environment {
        PATH = "/usr/local/bin:$PATH"
    }
    
    stages {

        stage('Common Config') {
            steps {
                echo 'Stage: Common Config'
                // Checkout repo with common config files/scripts to 'common' folder
                checkout([$class: 'GitSCM', 
                          branches: [[name: '*/main']], 
                          doGenerateSubmoduleConfigurations: false, 
                          extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'common']], 
                          submoduleCfg: [], userRemoteConfigs: [[url: 'file:///home/jenkins/gitrepos/cicd-common']]
                         ])
            }
        }
        stage('Ping Servers') {
            steps {
                echo 'Stage: Ping Servers'
                dir('common') {
                    sh '''
                    ./pingAll.sh
                    '''
                }
            }
        }
        stage('Get Testcases') {
            options {
                timeout(time: 1, unit: 'MINUTES')
            }
            steps {
                echo 'Stage: Get Testcases'
                dir('common') {
                    sh '''
                    ./tm_get_testcases.sh
                    cat ./tomahawk-tpid.txt
                    cat ./tomahawk-tcid.txt
                    wc -l ./tomahawk-tcid.txt
                    '''
                }
            }
        }
        stage('Run Each Testcase') {
            steps {
                echo 'Stage: Test Manager'
                dir('common') {
                    script {
                        def cases = readFile("tomahawk-tcid.txt").split("\\r?\\n");
                        cases.each { String testcase_name ->
                            echo "testcase: ${testcase_name}"
                            testcase_name = testcase_name.split(":")[1]
                            echo "TestName: ${testcase_name}"
                            build(job: '/TestMgrRunOne/main', parameters: [string(name: 'TestName', value: "${testcase_name}")], wait: true)
                        }
                    }
                }
                echo 'Stage: Test Manager complete'
            }
        }

    }
    post {
        always {
            echo "post/always"
            deleteDir() // clean workspace
        }
        success {
            echo "post/success"
        }
        failure {
            echo "post/failure"
        }
    }
}
