pipeline {
    agent any 

    stages {
        stage('Provision') { 
            steps {
                sh label: '', script: 'terraform init -input=false'
                sh label: '', script: 'terraform plan -out=tfplan -input=false'   
                sh label: '', script: 'terraform apply -input=false tfplan'
            }
        }
        stage('Giving EIPs') {
            steps {
        		sh label: '', script: '''chmod 400 docker.pem 
                callip=$(terraform show | grep public_ip | sed 's/"//g' | awk '{print $3}' | head -n2 | tail -n1)
                while ! ssh -i docker.pem -o StrictHostKeyChecking=no ec2-user@$callip uname &> /dev/null
        		do
            			printf "Connection Time Out"
                        sleep 30s
        		done
                echo "Success Get Response"
                sed -i "s/AWSIP/$callip/g" host.inv
        		'''
            }
        }
        stage('Installation Service') {
            steps {
                sh label: '', script: 'ansible-playbook update.yml -i host.inv'
            }
        }
        stage('Remove Workspace') {
            steps {
                sh label: '', script: 'rm -rf *'
            }
        }
    }

    post {
        always {
            emailext attachLog: true, attachmentsPattern: 'generatedFile.txt',
            body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
            recipientProviders: [developers(), requestor()],
            subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
            echo 'Clean Up Data'
            deleteDir()
        }
    }
}