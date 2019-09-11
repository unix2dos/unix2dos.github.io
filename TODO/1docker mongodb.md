mkdir ~/data

sudo docker pull mongo:latest 

sudo docker run -d -p 27017:27017 --name mongo -v /home/liuwei/data:/data/db mongo:latest

sudo docker exec -it mongo mongo









db.createCollection('cities') db.cities.insert({ name: 'New York', country: 'USA' }) db.cities.insert({ name: 'Paris', country: 'France' }) db.cities.find()







Migrating data to a new installation

Let's start a new MongoDB container, this time running on port 37017 instead of the default 27017:

\# Copy the data from the previous container sudo cp -r ~/data ~/data_clone  # Start another MongoDB container sudo docker run -d -p 37017:27017 -v ~/data_clone:/data/db mongo











https://www.thachmai.info/2015/04/30/running-mongodb-container/