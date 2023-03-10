pipeline {
    agent any
    parameters {
        choice(name: 'DST', choices: ['user1@localhost', 'user2@localhost'], description: 'Pick destination')
    }
    
    stages {

        stage("Preparation ") {
            steps {
                sh "mkdir ~/cert_backup_${currentBuild.number}/"
                sh "mkdir ~/cert_new_${currentBuild.number}/"
            }
        }

        stage("Connectivity check") {
            steps {
                sh "ssh ${DST} 'echo `hostname` `id`'"
            }
        }
        stage("Collect ") {
            steps {
                sh "scp ${DST}:~/test_cert/agent.crt ~/cert_backup_${currentBuild.number}/ || true"
                sh "scp ${DST}:~/test_cert/agent.key ~/cert_backup_${currentBuild.number}/ || true"
            }
        }
        stage("Generate Cert  ") {
            steps {
                sh "openssl genrsa -out ~/cert_new_${currentBuild.number}/${DST}.key 2048"
                sh "openssl req -new -key ~/cert_new_${currentBuild.number}/${DST}.key -out ~/cert_new_${currentBuild.number}/${DST}.csr -subj '/C=GB/ST=London/L=London/O=Global Security/OU=IT Department/CN=example.com'"
                sh "cat ~/cert_new_${currentBuild.number}/${DST}.csr"
                sh "scp -o 'StrictHostKeyChecking no' ~/cert_new_${currentBuild.number}/${DST}.csr gaixas1@hvisor:/home/gaixas1/csr/"
                sh "ssh -o 'StrictHostKeyChecking no' gaixas1@hvisor '~/gen_cert.sh ${DST}'"
                sh "scp -o 'StrictHostKeyChecking no'  gaixas1@hvisor:/home/gaixas1/crt/${DST}.crt ~/cert_new_${currentBuild.number}/${DST}.crt"
                sh "mv ~/cert_new_${currentBuild.number}/${DST}.key ~/cert_new_${currentBuild.number}/agent.key"
                sh "mv ~/cert_new_${currentBuild.number}/${DST}.crt ~/cert_new_${currentBuild.number}/agent.crt"
                sh "mv ~/cert_new_${currentBuild.number}/${DST}.csr ~/cert_new_${currentBuild.number}/agent.csr"
            }
        }
        stage("Check Keys ") {
            steps {
                script {
                    CERT = sh (script: "openssl x509 -noout -modulus -in ~/cert_new_${currentBuild.number}/agent.crt |openssl md5",returnStdout: true).trim()
                    KEY  = sh (script: "openssl rsa  -noout -modulus -in ~/cert_new_${currentBuild.number}/agent.key |openssl md5",returnStdout: true).trim()
                    if ( CERT == KEY ) {
                        echo "Key matches cert"
                    } else {
                        sh "exit 1"
                    }
                }
            }
        }
        stage("Publish ") {
            steps {
                sh "ssh ${DST} 'mkdir  ~/test_cert/cert_${currentBuild.number}'"
                sh "scp ~/cert_new_${currentBuild.number}/agent.csr ${DST}:~/test_cert/cert_${currentBuild.number}/"
                sh "scp ~/cert_new_${currentBuild.number}/agent.key ${DST}:~/test_cert/cert_${currentBuild.number}/"
                sh "scp ~/cert_new_${currentBuild.number}/agent.crt ${DST}:~/test_cert/cert_${currentBuild.number}/"
                                
                sh "ssh ${DST} 'ln -sf ~/test_cert/cert_${currentBuild.number}/agent.key ~/test_cert/agent.key'"
                sh "ssh ${DST} 'ln -sf ~/test_cert/cert_${currentBuild.number}/agent.crt ~/test_cert/agent.crt'"
            }
        }
    }

    post { 
        always { 
            sh "rm -R ~/cert_backup_${currentBuild.number}"
            sh "rm -R ~/cert_new_${currentBuild.number}"
        }
    }
}