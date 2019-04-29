# govuk-mongodb-content
Use Docker to run a local MongoDB instance containing GOV.UK content store and inspect with Mongo Express.

## Get Docker
Download and install Docker. If unfamiliar check out their [tutorials](https://docs.docker.com/get-started/) before proceeding.  

Check your Docker works by running the following code in your terminal:  

```
docker run hello-world
```

Get help with docker with:  

```
docker help
```

## Docker and MongoDB

### Get the Image and run the container

The first time you run the following code the Image will need to be downloaded; go make a cup of tea.  

```
docker run --name govuk -d -v /tmp/mongodb:/data/db -p 27017:27017 hello-mongo
```

Where the arguments are:  

```
--name: Name of the container.  
-d: Will start the container as a background (daemon) process. Don’t specify this argument to run the container as foreground process.  
-v: Attach the /tmp/mongodb volume of the host system to /data/db volume of the container.  
-p: Map the host port to the container port.
hello-mongo: Last argument is the name/id of the image. The version can be specified for reproducibility with a colon.    
```

### Prepare data
Get the content store database dump from a fellow GOV.UK data scientist or developer. Move your data dump of the content store, something like, `content_items.bson` into `/tmp/mongodb`.  

Your data should be in a directory that your container can now access.  

### Check for data
Open a bash shell in your recently spun-up govuk container with:

```
docker exec -it govuk bash
```

Check that you can see `content_items.bson` in the correct directory (i.e. the container can access your local volume specified above and the files therein).

### Restore MongoDB from .bson

From the bash shell of the container run:

```
mongorestore -d content_store -c content_items db/content_items.bson
```

This should start restoring the `content_items` collection in the `content_store` database on the MongoDB instance.  

We can access the MongoDB instance from the container with:

```
mongo 27017
```
Type help for help in the mongo shell.

Run some of these commands to test the database is there.

### Interact with MongoDB instance

Open a new terminal so we can link to it from another container. We download and build a new container from the image `mongo-express` which provides a user interface for managing your MongoDB databases.  

```
docker run --link govuk:mongo -p 8081:8081 mongo-express
```

In your browser, you can look at it for sense check by:

```
http://0.0.0.0:8081/
```

### Leveraging the MongoDB with pymongo
We can connect to and make use of this data for metadata extraction and other uses. In these [notebooks](https://github.com/alphagov/govuk-network-embedding/tree/master/notebooks/db) we extract the structural network, or those pages that have embedded links on each page.  
