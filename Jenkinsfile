def DST_LIST = [
    "user1@localhost",
    "user2@localhost"
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
                            sh "scp -o 'StrictHostKeyChecking no'  gaixas1@hvisor:/home/gaixas1/crt/${DST}.crt ~/cert_new_${currentBuild.number}/${DST}/"
                        }
                        stage("Publish "+DST) {
                            echo "DST : ${DST}"
                        }
                    }
                }
            }
        }
    }
}