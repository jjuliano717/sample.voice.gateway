# Deploying IBM Voice Gateway on Bluemix Kubernetes cluster or Kubernetes on a VM or on IBM Cloud private
Running the IBM Voice Gateway in a Kubernetes environment requires special considerations beyond the deployment of a typical HTTP based application. Because the voice gateway relies on SIP for call signaling and RTP for media which require affinity to a specific voice gateway instance, the Kubernetes ingress router needs to be avoided for these protocols. To work around the limitations of the ingress router, the voice gateway containers must be configuered in net Host mode, which means that when a port is opened in either of the voice gateway containers, those identical ports are also opened and mapped on the base VM. This also eliminates the need to define media port ranges in the kubectl configuration file which is not currently supported.

In Kubernetes terminology, a single voice gateway instance equates to a single Pod which contains both a SIP Orchestrator container and a Media Relay container. Only one POD should be deployed per node and the POD should be deploy with hostNetwork set to true. This will ensure that the SIP and media ports would be opened on the host VM and visible by the SIP Load Balancer.  

## Script defaults:

* Enforces one POD per node. If replicas (PODs > #nodes, the extra replica to be scheduled will remain in a waiting state)
* Exposes SIP and media relay ports on the associated VM by setting hostNetwork to true
* Auto restart of any failed containers

# Deploying Voice Gateway in multi-tenant mode:

1) Configure properties in tenantConfig.json before deployment. 
   - For more information - [Configuring multi-tenancy](https://www.ibm.com/support/knowledgecenter/SS4U29/multitenancy.html)

1) Create secret called tenantconfig from the file tenantConfig.json:
   ```bash
   kubectl create secret generic tenantconfig --from-file=tenantConfig=tenantConfig.json
   ```

1) If you want to enable recording (Optional): 
   - Set ENABLE_RECORDING to true in deploy.yaml and create the recording [PersistentVolume and PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) using the recording-pv.yaml and recording-pvc.yaml templates.
   - Update the recording template files  and run the following commands: 
     ```bash
     kubectl create -f recording-pv.yaml
     kubectl create -f recording-pvc.yaml
     ```
   - Uncomment recording volume and volumeMounts sections of the deploy-multitenant.yaml

1) If you want to use MRCPv2 config file (Optional):
   - More info: [Configuring services with MRCPv2](https://www.ibm.com/support/knowledgecenter/SS4U29/MRCP.html)
   - Create unimrcpConfig secret from the unimrcpclient.xml file using the following command: 
     ```bash
     kubectl create secret generic unimrcp-config-secret --from-file=unimrcpConfig=unimrcpclient.xml
     ```
   - Uncomment the unimrcpconfig volume and volumeMounts sections of the deploy-multitenant.yaml 
  
1) Deploy Voice Gateway:  
   ```bash
   kubectl create -f deploy-multitenant.yaml
   ```