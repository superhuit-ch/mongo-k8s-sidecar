# Mongo Kubernetes Replica Set Sidecar

This project is as a PoC to setup a mongo replica set using Kubernetes. It should handle resizing of any type and be 
resilient to the various conditions both mongo and kubernetes can find themselves in.

## How to use it

The docker image is hosted on docker hub and can be found here:  
https://registry.hub.docker.com/u/leportlabs/mongo-k8s-sidecar/

An example kubernetes replication controller can be found in the examples directory on github here:  
https://github.com/leportlabs/mongo-k8s-sidecar

There you will also find some helper scripts to test out creating the replica set and resizing it.

## Authentication
If this is your first setup, please remove the "--auth" from the apropriated mongo-controller yaml template
at ./example/mongo-controller-xxxxx-template.yaml

To add authentication simply follow the MongoDB documentation to create a new admin user and then restore the "--auth"
option here. You'll be then able to delete your replicas and add them back.
(!) This method to enable authentication requires that you have a persistent storage attached to the DB (!)

### Settings

- MONGO_SIDECAR_POD_LABELS  
  Required: YES  
  This should be a be a comma separated list of key values the same as the podTemplate labels. See above for example.
- MONGO_SIDECAR_SLEEP_SECONDS  
  Required: NO  
  Default: 5  
  This is how long to sleep between work cycles.
- MONGO_SIDECAR_UNHEALTHY_SECONDS  
  Required: NO  
  Default: 15  
  This is how many seconds a replica set member has to get healthy before automatically being removed from the replica set.

## Debugging

TODO: Instructions for cloning, mounting and watching

## Still to do

- Add tests!
- Add to circleCi
- Alter k8s call so that we don't have to filter in memory