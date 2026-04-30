pipeline {
    agent any

    environment {
        SF_CONSUMER_KEY = credentials('salesforce-consumer-key')
        SF_USERNAME     = credentials('salesforce-username')
        SF_JWT_KEY_FILE = credentials('salesforce-server-key')
    }

    stages {
        stage('Checkout del Codice') {
            steps {
                echo 'Scaricando il codice e la cronologia da Git...'
                checkout scm
            }
        }

        stage('Autenticazione Salesforce') {
            steps {
                echo 'Autenticazione in corso...'
                bat 'sf org login jwt --client-id %SF_CONSUMER_KEY% --jwt-key-file "%SF_JWT_KEY_FILE%" --username %SF_USERNAME% --set-default --alias myOrg'
            }
        }

        stage('Calcolo Delta') {
            steps {
                echo 'Analisi dei file modificati nell\'ultimo push...'
                // Genera la cartella "package" contenente il package.xml con le sole modifiche
                bat 'sf sgd source delta --to "HEAD" --from "HEAD~1" --output .'
                
                // Opzionale: Stampa a video i file che stanno per essere deployati per controllo
                bat 'type package\\package.xml'
            }
        }

        stage('Deploy Delta') {
            steps {
                echo 'Deploy dei metadati differenziali verso Salesforce...'
                // Usiamo --manifest invece di fare il deploy di tutto il progetto
                bat 'sf project deploy start --manifest package/package.xml --target-org myOrg --ignore-conflicts --wait 30'
            }
        }
    }

    post {
        always {
            echo 'Pulizia dell\'ambiente di lavoro...'
            cleanWs()
        }
        success {
            echo 'Deploy differenziale completato con successo! 🚀'
        }
        failure {
            echo 'Deploy fallito. Controlla i log per i dettagli sull\'errore. ❌'
        }
    }
}