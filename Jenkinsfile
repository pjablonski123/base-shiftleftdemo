node {
    files= ['deploy.yml']

    withCredentials([usernamePassword(credentialsId: 'prisma_cloud', passwordVariable: 'PC_PASS', usernameVariable: 'PC_USER')]) {
    PC_TOKEN = sh(script:'''curl -s -k -H 'Content-Type: application/json' -H 'accept: application/json' --data '{\"username\":\"$PC_USER\", \"password\":\"$PC_PASS\"}' https://${AppStack} | jq --raw-output .token''', returnStdout:true).trim()
    }

    stage('Clone repository') {
        checkout scm
    }

	

    stage('Check image Git dependencies has no vulnerabilities') {
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
    stage('Apply security policies (Policy-as-Code) for evilpetclinic') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh('chmod +x files/addPolicies.sh && ./files/addPolicies.sh')
        }
    }

    //$PC_USER,$PC_PASS,$PC_CONSOLE when Galileo is released. 
    stage('Download latest twistcli') {
        withCredentials([usernamePassword(credentialsId: 'prisma_cloud', passwordVariable: 'PC_PASS', usernameVariable: 'PC_USER')]) {
            sh 'curl -k -u $PC_USER:$PC_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
            sh 'chmod a+x ./twistcli'
        }
    }

    stage('Scan image with twistcli') {
            sh 'cd /var/jenkins_home/workspace/shiftleftdemo'
	    //sh 'docker pull solalraveh/evilpetclinic'
	    sh 'docker pull nginx'
	        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh('./twistcli images scan --u $TL_USER --p $TL_PASS --address https://$TL_CONSOLE --details solalraveh/evilpetclinic')
	//sh('./twistcli images scan --u $TL_USER --p $TL_PASS --address https://$TL_CONSOLE --details nginx')
        }
	    
	    // Scan the image
            //prismaCloudScanImage ca: '',
            //cert: '',
            //dockerAddress: 'unix:///var/run/docker.sock',
            //image: 'solalraveh/evilpetclinic:latest',
            //key: '',
            //logLevel: 'info',
            //podmanPath: '',
            //project: '',
            //resultsFile: 'prisma-cloud-scan-results.json',
            //ignoreImageBuildTime:true
	    
stage("Scan Cloud Formation Template with API v2") {
	sh 'chmod a+x ./twistcli'
        

    }
}
}


