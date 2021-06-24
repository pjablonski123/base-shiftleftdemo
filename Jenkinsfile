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

        def response


        response = sh(script:"curl -sq -X POST -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans --data-binary '@scan-asset.json'", returnStdout:true).trim()

        def SCAN_ASSET = readJSON text: response

        def SCAN_ID = SCAN_ASSET['data'].id
        def SCAN_URL = SCAN_ASSET['data']['links'].url

        //Upload files
        sh(script:"curl -sq -X PUT  --url '${SCAN_URL}' -T 'files/deploy.yml'", returnStdout:true).trim()

        //start the Scan
        response = sh(script:"curl -sq -X POST -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans/${SCAN_ID} --data-binary '@scan-start-k8s.json'", returnStdout:true).trim()


        //Get the Status
        def SCAN_STATUS
        def STATUS

        //Need a Do-While loop here.   Haven't found a good syntax with Groovy in Jenkins
        response = sh(script:"curl -sq -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans/${SCAN_ID}/status", returnStdout:true).trim()
        SCAN_STATUS = readJSON text: response
        STATUS = SCAN_STATUS['data']['attributes']['status']

        while  (STATUS == 'processsing'){
            response = sh(script:"curl -sq -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans/${SCAN_ID}/status", returnStdout:true).trim()
            SCAN_STATUS = readJSON text: response
            STATUS = SCAN_STATUS['data']['attributes']['status']
            print "${STATUS}"

        }

        //Get the Results
        response = sh(script:"curl -sq -H 'x-redlock-auth: ${PC_TOKEN}' -H 'Content-Type: application/vnd.api+json' --url https://${AppStack}/iac/v2/scans/${SCAN_ID}/results", returnStdout:true).trim()
        def SCAN_RESULTS= readJSON text: response

        print "${SCAN_RESULTS}"

}

    }
}



