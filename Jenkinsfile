def DST_LIST = [
    "user1@localhost"
]

pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                sh "mkdir ~/cert_backup_${currentBuild.number}"
                sh "mkdir ~/cert_new_${currentBuild.number}"
                script {
                    DST_LIST.each { DST ->
                        stage("Connectivity check "+DST) {
                            sh "ssh ${DST} 'echo `hostname` `id`'"
                        }
                        stage("Collect "+DST) {
                            sh "mkdir ~/cert_backup_${currentBuild.number}/${DST}"
                            sh "scp ${DST}:~/test_cert/agent.crt ~/cert_backup_${currentBuild.number}/${DST}/ || true"
                            sh "scp ${DST}:~/test_cert/agent.key ~/cert_backup_${currentBuild.number}/${DST}/ || true"
                            
                        }
                        stage("Generate Cert  "+DST) {
                            sh "mkdir ~/cert_new_${currentBuild.number}/${DST}"
                            sh "openssl genrsa -out ~/cert_new_${currentBuild.number}/${DST}/${DST}.key 2048"
                            sh "openssl req -new -key ~/cert_new_${currentBuild.number}/${DST}/${DST}.key -out ~/cert_new_${currentBuild.number}/${DST}/${DST}.csr -subj '/C=GB/ST=London/L=London/O=Global Security/OU=IT Department/CN=example.com'"
                            sh "cat ~/cert_new_${currentBuild.number}/${DST}/${DST}.csr"
                            sh "scp -o 'StrictHostKeyChecking no' ~/cert_new_${currentBuild.number}/${DST}/${DST}.csr gaixas1@hvisor:/home/gaixas1/csr/"
                            sh "ssh -o 'StrictHostKeyChecking no' gaixas1@hvisor '~/gen_cert.sh ${DST}'"
                            sh "scp -o 'StrictHostKeyChecking no'  gaixas1@hvisor:/home/gaixas1/crt/${DST}.crt ~/cert_new_${currentBuild.number}/${DST}/${DST}.crt"
                            sh "mv ~/cert_new_${currentBuild.number}/${DST}/${DST}.key ~/cert_new_${currentBuild.number}/${DST}/agent.key"
                            sh "mv ~/cert_new_${currentBuild.number}/${DST}/${DST}.crt ~/cert_new_${currentBuild.number}/${DST}/agent.crt"
                            sh "mv ~/cert_new_${currentBuild.number}/${DST}/${DST}.csr ~/cert_new_${currentBuild.number}/${DST}/agent.csr"
                        }
                        stage("Check Keys "+DST) {
                            CERT = sh (script: "openssl x509 -noout -modulus -in ~/cert_new_${currentBuild.number}/${DST}/agent.crt |openssl md5",returnStdout: true).trim()
                            KEY  = sh (script: "openssl rsa  -noout -modulus -in ~/cert_new_${currentBuild.number}/${DST}/agent.key |openssl md5",returnStdout: true).trim()
                            if ( CERT == KEY ) {
                                echo "Key matches cert"
                            } else {
                                sh "exit 1"
                            }
                        }
                        stage("Publish "+DST) {
                            sh "ssh ${DST} 'mkdir  ~/test_cert/cert_${currentBuild.number}'"
                            sh "scp ~/cert_new_${currentBuild.number}/${DST}/agent.csr ${DST}:~/test_cert/cert_${currentBuild.number}/"
                            sh "scp ~/cert_new_${currentBuild.number}/${DST}/agent.key ${DST}:~/test_cert/cert_${currentBuild.number}/"
                            sh "scp ~/cert_new_${currentBuild.number}/${DST}/agent.crt ${DST}:~/test_cert/cert_${currentBuild.number}/"
                            
                            sh "ssh ${DST} 'ln -sf ~/test_cert/cert_${currentBuild.number}/agent.key ~/test_cert/agent.key'"
                            sh "ssh ${DST} 'ln -sf ~/test_cert/cert_${currentBuild.number}/agent.crt ~/test_cert/agent.crt'"
                        }
                    }
                }
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