CREATE_SERVICE=TRUE

# ENV Options
# -GoogleCloudPlatform
# -FLOCKERAWS
# -AWS (NOT YET SUPPORTED)
# -GlusterFS
ENV=GlusterFS

# Default volume size on creation (this only works for GCE)
CREATE_DISK=false
DELETE_DISK=false
DISK_SIZE=100GB
DISK_TYPE=pd-standard

# GoogleCloudPlatform Specific Environment Variables
ZONE=europe-west1-d

# ENVIRONMENT for labels
ENVIRONMENT=test

# NAMESPACE (default, production, etc.)
NAMESPACE=test
CREATE_NAMESPACE=true

# MONGODB VERSION which docker image to use.
DBVERSION=3.3

# FIND REPLICAS
NUM_REPLICAS=$(shell kubectl get rc --namespace=$(NAMESPACE) -l role=mongo,environment=$(ENVIRONMENT) -o template --template='{{ len .items }}')
NEW_REPLICA_NUM=$(shell expr $(NUM_REPLICAS) + 1 )

# Create GCE PD disks ?

# FLOCKERAWS Specific Environment Variables
CONTROL_DNS=<FLOCKER CONTROL DNS>
INITIAL_NODE_ID=<A FLOCKER NODE ID TO SEED A VOLUME>
VOLUME_PREFIX=<any name to prefix your volumes with>
KUBECONFIG=clusters/<K8s Cluster Name>/kubeconfig

########## Count number of replicas
count:
	@echo 'Current Number of MongoDB Replicas on environment $(ENVIRONMENT) and namespace $(NAMESPACE): $(NUM_REPLICAS)'


########## CREATE FLOCKER VOLUME
create-volume:
ifeq ($(ENV),FLOCKERAWS)
	-flockerctl --control-service=$(CONTROL_DNS) create -m name=$(VOLUME_PREFIX)-$(ENVIRONMENT)-$(NEW_REPLICA_NUM) -s $(DISK_SIZE) --node=$(INITIAL_NODE_ID)
endif


########## ADD A REPLICA
add-replica:
## NAMESPACE
ifeq ($(CREATE_NAMESPACE),true)
	@echo 'Creating namespace: $(NAMESPACE)'
	-sed -e 's/<nspace>/$(NAMESPACE)/g' create-namespace.yaml | kubectl create -f -
endif
## GCE
ifeq ($(ENV),GoogleCloudPlatform)
ifeq ($(CREATE_DISK),true)
	@echo 'Creating Disk for environment: $(ENVIRONMENT) and namespace: $(NAMESPACE)'
	-gcloud compute disks create mongo-persistent-storage-node-$(ENVIRONMENT)-$(NEW_REPLICA_NUM)-disk --size $(DISK_SIZE) --zone $(ZONE) --type $(DISK_TYPE)
endif
	@echo 'Adding Replica to environment: $(ENVIRONMENT) and namespace: $(NAMESPACE)'
	-sed -e 's/<num>/$(NEW_REPLICA_NUM)/g' -e 's/<env>/$(ENVIRONMENT)/g' -e 's/<dbver>/$(DBVERSION)/g' -e 's/<nspace>/$(NAMESPACE)/g' mongo-controller-template.yaml | kubectl create -f -
endif
## GlusterFS
ifeq ($(ENV),GlusterFS)
	@echo 'Adding Replica to environment: $(ENVIRONMENT) and namespace: $(NAMESPACE)'
	-sed -e 's/<num>/$(NEW_REPLICA_NUM)/g' -e 's/<env>/$(ENVIRONMENT)/g' -e 's/<dbver>/$(DBVERSION)/g' -e 's/<nspace>/$(NAMESPACE)/g' mongo-controller-glusterfs-template.yaml | kubectl create -f -
endif
## AWS
ifeq ($(ENV),AWS)
	@echo 'AWS not supported yet'
endif
## Flocker on aws
ifeq ($(ENV),FLOCKERAWS)
	@echo 'Adding Replica $(NEW_REPLICA_NUM) to environment: $(ENVIRONMENT) and namespace: $(NAMESPACE)'

	-touch mongo-rc-$(ENVIRONMENT)-$(NEW_REPLICA_NUM).yaml
	# replace volume name with declared $(VOLUME_PREFIX) and $(NEW_REPLICA_NUM)
	# This will create a new file allowing you to remove and redeploy specific Replication Controllers
	# kubectl create -f mongo-rc-test-1.yaml
	# kubectl delete -f mongo-rc-test-1.yaml
	-sed -e 's/<num>/$(NEW_REPLICA_NUM)/g' -e 's/<env>/$(ENVIRONMENT)/g' -e 's/<dbver>/$(DBVERSION)/g' -e 's/<nspace>/$(NAMESPACE)/g' -e 's/VOLUME_PREFIX/$(VOLUME_PREFIX)/g' mongo-controller-flocker-template.yaml | tee mongo-rc-$(ENVIRONMENT)-$(NEW_REPLICA_NUM).yaml
	-kubectl create -f mongo-rc-$(ENVIRONMENT)-$(NEW_REPLICA_NUM).yaml
endif
ifeq ($(CREATE_SERVICE),TRUE)
	@echo 'Creating Service to environment: $(ENVIRONMENT) and namespace: $(NAMESPACE)'
	-sed -e 's/<num>/$(NEW_REPLICA_NUM)/g' -e 's/<env>/$(ENVIRONMENT)/g' -e 's/<dbver>/$(DBVERSION)/g' -e 's/<nspace>/$(NAMESPACE)/g' mongo-service-template.yaml | kubectl create -f -
endif


########## DELETE REPLICA
delete-replica:
	@echo 'Deleting Service on environment: $(ENVIRONMENT) and namespace: $(NAMESPACE)'
	-kubectl delete svc mongo-$(ENVIRONMENT)-$(NUM_REPLICAS) --namespace=$(NAMESPACE)
ifeq ($(ENV),GlusterFS)
	@echo 'Deleting Replic on environment: $(ENVIRONMENT) and namespace: $(NAMESPACE)'
	-kubectl delete rc mongo-$(ENVIRONMENT)-$(NUM_REPLICAS) --namespace=$(NAMESPACE)
endif
ifeq ($(ENV),GoogleCloudPlatform)
	@echo 'Deleting Replic on environment: $(ENVIRONMENT) and namespace: $(NAMESPACE)'
	-kubectl delete rc mongo-$(ENVIRONMENT)-$(NUM_REPLICAS) --namespace=$(NAMESPACE)
ifeq ($(DELETE_DISK),true)
	@echo 'Deleting Disk on environment: $(ENVIRONMENT)'
	sleep 60
	-yes | gcloud compute disks delete mongo-persistent-storage-node-$(ENVIRONMENT)-$(NUM_REPLICAS)-disk --zone $(ZONE)
endif
endif
ifeq ($(ENV),AWS)
	@echo 'AWS not supported yet'
endif
