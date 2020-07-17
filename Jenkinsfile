pipeline{
    agent{
        label "node"
    }
    triggers {
        cron 'H 20 * * *'
    }
    options {
        timeout(time: 5, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuild()
    }
    stages{
        stage("Backup GitHub"){
            // Trigger only by timer and human, but not by Upstream or scm commit
            when {
                beforeAgent true
                expression { BRANCH_NAME ==~ /(master|main)/ }
                anyOf {
                    triggeredBy cause: 'UserIdCause'
                    triggeredBy cause: 'TimerTrigger'
                }
            }
            steps{
                echo "========executing A========"
                sh """
                    ./bin/ghe-backup -v | tee backup${BUILD_NUMBER}.log
                    grep -q "Completed backup" backup${BUILD_NUMBER}.log || exit 1
                """
            }
        }
    }
    post{
        failure {
            echo "========email========"
            mail to: 'vs.sagar@gmail.com'
            subject: "GitHub Daily backup Failed: ${currentBuild.fullDisplayName}",
            body: "This is important, can you please check and bring it to team's notice? ${env.BUILD_URL}"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}