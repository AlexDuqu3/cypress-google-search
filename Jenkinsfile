pipeline {
        agent any

        tools {nodejs "NodeJS"}


        parameters{
            string(name: 'SPEC', defaultValue:"cypress/integration/google-search.js", description: "Enter the cypress script path that you want to execute")
            choice(name: 'BROWSER', choices:['electron'], description: "Select the browser to be used in your cypress tests")
        }

        options {
                ansiColor('xterm')
        }

        
        stages {
        stage('Run automated tests'){
            steps {
                echo "Running automated tests"
                sh 'npm prune'
                sh 'npm cache clean --force'
                sh 'npm i'
                sh 'npm install --save-dev mochawesome mochawesome-merge mochawesome-report-generator'
                //sh 'npm run e2e:staging1spec'
            }
            /*post {
                success {
                    publishHTML (
                        target : [
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'mochawesome-report',
                            reportFiles: 'mochawesome.html',
                            reportName: 'My Reports',
                            reportTitles: 'The Report'])

                }
            }*/
        }

        stage('SonarQube analysis') {
            steps {
            script {
                        scannerHome = tool 'sonar-scanner';
                    }
            withSonarQubeEnv('GoogleSearch') { // If you have configured more than one global server connection, you can specify its name
            sh "${scannerHome}/bin/sonar-scanner"
            }
            }
        }
        
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('JMeter Test') {
            when { expression { params.skip_jmeter != true } } 
            steps {
                script {
                    // Path to the JMeter installation directory
                    def jmeterHome = '/usr/share/jmeter'

                    // Path to the JMeter test script
                    def jmeterScript = './jmeter.jmx'

                    // Execute JMeter test
                    sh "${jmeterHome}/bin/jmeter -n -t ${jmeterScript} -l result.jtl"
                }
            }
            post {
                always {
                    // Archive JTL result file
                    archiveArtifacts 'result.jtl'
                }
                success {
                    // Publish JMeter report using Performance plugin
                    perfReport filterRegex:'', sourceDataFiles: 'result.jtl'
                }
            }
        }


        stage('Perform manual testing...'){
            steps {
                timeout(activity: true, time: 48, unit: 'HOURS') {
                    input 'Proceed to production?'
                }
            }
        }

        stage('Release to production') {
            steps {
            // similar procedure as in the 'Build/ Deploy to staging' stage, suppressed here for cost saving purposes
                echo "Deploying app in production environment"
            }
        }
    }
}
