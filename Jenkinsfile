/*
This pipeline will carry out the following on the project:

1. Git secret checker
2. Software Composition Analysis
3. Static Application Security Testing
4. Container security audit 
5. Dynamic Application Security Testing
6. Host system security audit
7. Host application protection
*/

testenv = "null"

pipeline {
    /* Which agent are we running this pipeline on? We can configure different OS */
    agent any
	
    stages {   
        stage('Checkout project'){
            steps {
                echo 'downloading git directory..'
                git 'https://github.com/pawnu/secDevLabs.git'
            }
        }      
        stage('git secret check'){
            steps{
                script{
                    echo 'running trufflehog to check project history for secrets'
                    sh 'bash -c "source /opt/venv/bin/activate && trufflehog --regex --entropy=False --max_depth=3 https://github.com/pawnu/secDevLabs"'
                }
            }
        }
        stage('SCA'){
            steps{
                echo 'running python safety check on requirements.txt file'
                sh 'bash -c "source /opt/venv/bin/activate && safety check -r $WORKSPACE/owasp-top10-2017-apps/a7/gossip-world/app/requirements.txt"'
            }
        }  
        stage('SAST') {
            steps {
                echo 'Testing source code for security bugs and vulnerabilities'
                sh 'bash -c "source /opt/venv/bin/activate && bandit -r $WORKSPACE/owasp-top10-2017-apps/a7/gossip-world/app/ -ll || true"'
            }
        }
        stage('Container audit') {
            steps {
                echo 'Audit the dockerfile used to spin up the web application'
                script{				
                    def exists = fileExists '/var/jenkins_home/lynis/lynis'
                    if(exists){
                        echo 'lynis already exists'
                    }else{
                        sh 'bash -c "source /opt/venv/bin/activate && wget https://downloads.cisofy.com/lynis/lynis-2.7.5.tar.gz && tar xfvz lynis-2.7.5.tar.gz -C ~/ && rm lynis-2.7.5.tar.gz"'
                    }
                }
                dir("/var/jenkins_home/lynis"){  
                    sh 'bash -c "source /opt/venv/bin/activate && mkdir $WORKSPACE/$BUILD_TAG/ && ./lynis audit dockerfile $WORKSPACE/owasp-top10-2017-apps/a7/gossip-world/deployments/Dockerfile | ansi2html > $WORKSPACE/$BUILD_TAG/docker-report.html && mv /tmp/lynis.log $WORKSPACE/$BUILD_TAG/docker_lynis.log && mv /tmp/lynis-report.dat $WORKSPACE/$BUILD_TAG/docker_lynis-report.dat"'
                }
            }
        }	    
        stage('Setup test env') {
            steps {
                sh '''
                bash -c "source /opt/venv/bin/activate"
                #refresh inventory
                echo "[local]" > ~/ansible_hosts
                echo "localhost ansible_connection=local" >> ~/ansible_hosts
                echo "[tstlaunched]" >> ~/ansible_hosts
                
                tar cvfz /var/jenkins_home/pythonapp.tar.gz -C $WORKSPACE/owasp-top10-2017-apps/a7/ .

                ssh-keygen -t rsa -N "" -f ~/.ssh/psp_ansible_key || true
                ansible-playbook -i ~/ansible_hosts ~/createAwsEc2.yml"
                '''		  
                script{
                    testenv = sh(script: "bash -c \"source /opt/venv/bin/activate && sed -n '/tstlaunched/{n;p;}' /var/jenkins_home/ansible_hosts\"", returnStdout: true).trim()
                }
                echo "${testenv}"
                sh 'bash -c "source /opt/venv/bin/activate && ansible-playbook -i ~/ansible_hosts ~/configureTestEnv.yml"'
            }
        }
        stage('DAST') {
            steps {
                script{				
                    //Test the web application from its frontend
                    def seleniumIp = env.SeleniumPrivateIp
                    if("${testenv}" != "null"){
                        sh 'bash -c "source /opt/venv/bin/activate && python ~/authDAST.py $seleniumIp ${testenv} $WORKSPACE/$BUILD_TAG/DAST_results.html"'
                    }  			
                }
            }
        }
        stage('System security audit') {
            steps {
                echo 'Run lynis audit on host and fetch result'
                sh 'bash -c "source /opt/venv/bin/activate && ansible-playbook -i ~/ansible_hosts ~/hostaudit.yml --extra-vars \"logfolder=$WORKSPACE/$BUILD_TAG/\""'
            }
        }
        stage('Deploy WAF') {
            steps {
                echo 'Deploy modsecurity as reverse proxy'
                sh 'bash -c "source /opt/venv/bin/activate && ansible-playbook -i ~/ansible_hosts ~/configureWAF.yml"'
            }
        }	    
    }
    post {
        always {
            echo 'We could bring down the ec2 here'
            /*
            echo 'Tear down activity'
            script{
                if("${testenv}" != "null"){
                    echo "killing host ${testenv}"
                    sh 'bash -c "source /opt/venv/bin/activate && ansible-playbook -i ~/ansible_hosts ~/killec2.yml"'
                } 
            }
	    }
     }
}
     
            */
