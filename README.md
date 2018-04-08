A simple CouchDB demo.

This tiny demo illustrates the use of a Couchbase (CB) server, the Sync Gateway (SG) and an Android app to retrieve values from a bucket. Conceptually the code consists of a Python3-based web app using Flask to offer a number of images on a web page. Clicking one of the images commits its name and bitmap to a predefined CB bucket.  Using CB's  SG, the Android app pulls the name and initial bitmap from the bucket upon startup and watches through a ChangeDelegate (a CouchbaseLite (CBL) object) for any changes. Should the bucket content change, the image and name on the Android screen is automatically updated.

I created this sample code to provide a shorter learning curve than the examples on the CB website which in my opinion require too much background knowledge for the absolute beginner who's just starting with the overall architecture and surrounding software ecosystem of this popular NoSQL database.

Caveat: This code comes as it is. No effort has been made  finish and polish this code base for production deployment (for example, error checking and logging is virtually non-existent). Also commenting could be vastly improved. Proceed at your own risk :-) and enjoy learning and playing with it.

Prerequisites for running this demo which worked for me:

* A
* B
* C

1. Install CB as a Docker container as described on the CB website (https://developer.couchbase.com/documentation/server/current/install/getting-started-docker.html)
2. Install SG as a Docker container as described on https://hub.docker.com/r/couchbase/sync-gateway ensuring to map the port range 4984-4985 instead of just the port 4984 (trust me, you'll need this later on for debugging as the admin port 4985 allows easy access to the synced buckets). In the "config" directory of the repo you will find a suitable SG configuration file - just make sure to map it to a location using a Docker volume where the SG running inside the container can find it or copy it into the container using "docker cp". This configuration contains a mapping to the server bucket and user credentials for logging into the CB server instance (see below). I purposely omitted any fancy  Javascript sync functionality as part of the SG configuration in order to keep it short and sweet. You can check if the SG is running correctly by opening the web page at port 4984 (using a browser or curl). If configured correclty, the SG should reply with a status JSON document indicating the version number, etc.
1. Install PIP (if you haven't done this yet :-) for Python3. Using PIP, install Flask and its dependencies. The CB Python client is called couchbase and can be installed using PIP as well. These steps can be done in a virtualenv or on bare metal depending on your preferences.
3. In the newly created CB server instance, create a user called "couchbase" with password "couchbase" with full admin rights to reflect the SG config settings explained above.
4. Create a bucket called "default" using "Couchbase" as the bucket type and a small memory footprint (as we will be storing only one document)
5. Install the Python Flask server code at a location of your choice and populate the "static" subdirectory with sample images, As the sample app represents a fashion blog (where a blogger can choose a picture which is then stored in the CB bucket and synchronized to the Android app running on the mobile), I chose images comparable to Ugly Dolls :-) but it goes without saying that these images are **not** part of the repo due to copyright reasons. Any images roughly square which can be rendered nicely on a web page will do (I chose approximately 500x500 px sizes). The current code base only recognizes ".png" files; this can easily be extended by expanding the "exts" list the function "getImages" in the server app called "app,py".
6. Once you run the Flask app, you should be greeted with a nice webpage offering the images you copied into the "static" folder. Clicking on an image will store the bitmap from the image file alonside the filename minus its extension in the bucket. You can check this by logging into the CB web UI and listing the documents of the "default" bucket.
7. If the SG is configured correctly, the synced document should now be available on SG server instance on port 4985 with the URL suffix _admin/db/db/documents/fashionPic ("fashionPic" being the document ID).
8. Using your Android / Java IDE of choice from IntelliJ :-) (I used AndroidStudio), open the project in the main repo folder. Verify in the IDE in the project configuration that both couchbase-lite-android and org.apache.commons.io are listed as library dependencies (otherwise your Android project won't build).
9. Modify the SG IP address in the member function "setupDB" to reflect your SG server instance. The database name "db" is reflected in the SG configuration JSON file - if you change this, please ensure that you change it both in the Java code as well as JSON config file (and don't forget to restart the SG Docker container :-).


Some hints:

- As the CB server logs as obtained by Docker are somewhat brief, it's worth checking out the logs inside the container. You will find them at /opt/couchbase/var/lib/couchbase/logs if you are using the standard CB image found on the Docker hub.
- Without a channel property a document will ***not*** be synchronized between the server and the gateway. This is why I included a "default" channel property in the document when creating it on the server side.
- The SG logs in contrast are quite comprehensive. docker logs <container name\> will do the trick.
- And - as usual - logcat or similar mechanisms (IDE-based or not) on the Android side provide you with a view from the client side.

I hope this code serves you as a starting point to explore the opportunities of synchronization between CB instances - have fun exploring it as much as I had writing the code!
