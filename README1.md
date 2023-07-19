MONGODB
we use docker official image - mongo. Checkout dockerhub and findout more information on how to use this image - root password and username, environmental variables, etc

# Use root/example as user/password credentials
version: '3.1'

services:

  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/

      
      

We will create secret and configMap to hold the confidential and sensitive information

Storing data in a Secret does not automatically make it secure. There are build it in mechanism (like encription) for basic security, which are not enabled by default.

run this formular in your terminal to generate the encoded values to be used in creating your secret.
echo -n 'username' | base64      #replace with username

echo -n 'password' | base64      #replace with password


NOTE:
=====
Order of creating the different deployment files mater alot. If we are to use the secret in our deployment, it should be created/existing before our pod deployment.
ConfigMap should already be running before mongodb express is deployed


MONGODB-EXPRESS
===============
We checked mongode-exp-ress on dockerhub to get the information we need in our mongodb deployment 
Mogodb-express will need the following:

Which database to connect to?
MongoDB Address / Internal Service
Which credentials to Authenticate?

See the information under configurations
ME_CONFIG_MONGODB_ADMINUSERNAME | ''              | MongoDB admin username
ME_CONFIG_MONGODB_ADMINPASSWORD | ''              | MongoDB admin password
ME_CONFIG_MONGODB_SERVER        | 'mongo'         | MongoDB container name. Use comma delimited list of host names for replica sets. 


docker run --network some-network -e ME_CONFIG_MONGODB_SERVER=some-mongo -p 8081(containerport):8081 mongo-express

The username and password will be same as the mongodb which is stored in our Secret. We will reference
it in the mongobd-express deployment

Two or more applications can use the same Secret /ConfigMap
If there is need for update in future it can be done in a single place

These are the 3 environmental variables mogo-express need to connect to our database -mongodb

 env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME 
          valueFrom: 
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD  
          valueFrom: 
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER  
          valueFrom: 
            configMapKeyRef:
              name: mongodb-configmap
              key: database_url
