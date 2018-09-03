stage("Checkout"){
    checkout scm
}
stage('Test ansible read credentials') {
    docker.image('williamyeh/ansible:centos7').inside() {
        withCredentials([string(credentialsId: 'test-credential-secret-text',
                variable: 'secret_text')]) {
            sh "ls -al"

            echo "${secret_text}"
            sh "ansible-playbook -i jenkins_local.ini " +
                    "-e 'secret_variable=${secret_text} normal_variable=123456' " +
                    "test_security.yml"
        }
    }
}