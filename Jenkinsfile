podTemplate(yaml : '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: centos
    image : centos
    command:
    - sleep
    args:
    - 99d
  restartPolicy: Never
''') {
  node(POD_LABEL) {

    git url: 'https://github.com/kamalakar07/week09.git', branch: env.BRANCH_NAME
    
    container('centos') {
			stage('preprocess') {
				sh '''
				yum install -y jq
				'''
			}
			stage('Test calculator updates') {
				
				
                //If calcservice is already running, check if the division functionailty exists.
                try{
                    sh '''
				    NAMESPACE=staging
                    #set CALCIP value
                    CALCIP=$(curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces/$NAMESPACE/services?fieldSelector=metadata.name%3Dcalculator-service | jq -r .items[0].spec.clusterIP)                  
                    # test calculator for division before image update
                    echo testing calculator for div before image upgrade. This should give 404 code if div functionailty does not exist
                    curl -i $CALCIP:8080/div?a=10\\&b=2
                    '''
                } catch (e){

                }
                
                sh'''
                # Update calculator
                curl -k -v -XPUT -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"  -H "Accept: application/yaml" -H "Content-Type: application/yaml" -H "User-Agent: kubectl/v1.21.5 (darwin/amd64) kubernetes/aea7bba" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/apis/apps/v1/namespaces/$NAMESPACE/deployments/calculator-deployment?fieldManager=kubectl-client-side-apply --data-binary '@calculator.yaml'
                
                # Give time for deployments to be up
                sleep 60

                # test calculator for div AFTER image update
                echo testing calculator for div AFTER image upgrade.gives 200 or 404 status code based on if div functionality exists
                CALCIP=$(curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT/api/v1/namespaces/$NAMESPACE/services?fieldSelector=metadata.name%3Dcalculator-service | jq -r .items[0].spec.clusterIP)                  
                curl -i $CALCIP:8080/div?a=10\\&b=2
				'''
			}

    }

  }
}