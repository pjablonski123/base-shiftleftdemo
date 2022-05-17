node {
    files= ['deploy.yml']

    //withCredentials([usernamePassword(credentialsId: 'prisma_cloud', passwordVariable: 'PC_PASS', usernameVariable: 'PC_USER')]) {
    //PC_TOKEN = sh(script:"curl -s -k -H 'Content-Type: application/json' -H 'accept: application/json' --data '{\"username\":\"$PC_USER\", \"password\":\"$PC_PASS\"}' https://${AppStack}/login | jq --raw-output .token", returnStdout:true).trim()
    //}

    stage('Clone repository') {
        checkout scm
    }


    stage('Download latest twistcli') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh 'curl -k -v -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v22.01/util/twistcli'
            sh 'sudo chmod a+x ./twistcli'
        }
    }

    stage('CodeRepo scan') {
        try {
            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
                sh('chmod +x files/checkGit.sh && ./files/checkGit.sh')
            }
        } catch (err) {
            echo err.getMessage()
            echo "Error detected"
			throw RuntimeException("Build failed for some specific reason!")
        }
    }

    //$PC_USER,$PC_PASS,$PC_CONSOLE when Galileo is released. 
    stage('Apply security policies') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh('chmod +x files/addPolicies.sh && ./files/addPolicies.sh')
        }
    }

    

    stage('Scan image with twistcli') {
        try {
		sh 'docker pull pasqu4le/evilpetclinic:latest'
            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
                sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
                sh 'sudo chmod a+x ./twistcli'
                sh "./twistcli images scan --u $TL_USER --p $TL_PASS --address https://$TL_CONSOLE --details pasqu4le/evilpetclinic"
            }
        } catch (err) {
            echo err.getMessage()
            echo "Error detected"
			throw RuntimeException("Build failed for some specific reason!")
        }
    }
	
        stage('IaC Scan') {
		    sh "sudo apt-get update"
		    sh "sudo apt-get -y install python3-pip"
		    sh "pip3 install pipenv"
		//sh "pipenv --venv"
		//sh "rm -r /root/.local/share/virtualenvs/shiftleftdemo-4WP0SCa2"
                    sh "pipenv install"
                    sh "export PRISMA_API_URL=https://api.prismacloud.io"
		    sh "export LOG_LEVEL=WARNING"
                    sh "pipenv run pip install bridgecrew"
	            sh "pipenv run bridgecrew --directory iac --bc-api-key $BC_API --repo-id pjablonski123/base-shiftleftdemo"        
	}
	
/*
    stage('Sandboxing') {
	withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
	sh "./twistcli sandbox --address https://$TL_CONSOLE --analysis-duration 2m --u $TL_USER  --p $TL_PASS --output-file sandbox_out.json pasqu4le/evilpetclinic:latest"
	}
    }	  
	  

    stage('Scan K8s yaml manifest with Bridgecrew') {  
	withDockerContainer(image: 'bridgecrew/jenkins_bridgecrew_runner:latest') {
	sh "/run.sh $BC_API https://github.com/pasqua1e/shiftleft_demo-build/" 
	}
    }
	*/

    stage('Deploy evilpetclinic') {
        sh 'kubectl create ns evil --dry-run -o yaml | kubectl apply -f -'
        sh 'kubectl delete --ignore-not-found=true -f files/deploy.yml -n evil'
        sh 'kubectl apply -f files/deploy.yml -n evil'
        sh 'sleep 30'
    }

    stage('Run bad Runtime attacks') {
        sh('chmod +x ./files/runtime_attacks.sh && ./files/runtime_attacks.sh')
    }

    stage('Run bad HTTP stuff for WAAS to catch') {
        sh('chmod +x ./files/waas_attacks.sh && ./files/waas_attacks.sh')
    }
}
