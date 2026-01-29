pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('InfluxDB Setup') {
            steps {
                bat '''
                    docker stop influxdb 2>nul && docker rm influxdb 2>nul || echo "Cleanup"
                    docker run -d --name influxdb -p 8086:8086 ^
                        -e INFLUXDB_DB=jmeter influxdb:1.8
                    timeout /t 15 /nobreak >nul
                    docker exec influxdb influx -execute "CREATE DATABASE jmeter"
                '''
            }
        }
        stage('JMeter') {
            steps {
                bat '''
                    if exist logs rmdir /s /q logs
                    if exist html rmdir /s /q html
                    mkdir logs
                    mkdir html
                    mkdir html\\report
                    "C:\\Users\\ksivv\\OneDrive\\Desktop\\apache-jmeter-5.6.3\\bin\\jmeter.bat" -n -t API.jmx -l logs/results.jtl -e -o html/report
                '''
            }
        }
        stage('Reports') {
            steps {
                perfReport sourceDataFiles: 'logs/results.jtl'
                publishHTML(
                    target: [
                        reportDir: 'html/report',
                        reportFiles: 'index.html',
                        reportName: 'JMeter HTML Report',
                        keepAll: true,
                        alwaysLinkToLastBuild: true,
                        allowMissing: false
                    ]
                )
            }
        }
    }
    post {
        always {
            archiveArtifacts 'logs/results.jtl, html/report/**'
        }
    }
}
