# We are going to deployment Drupal on GAE i.e. Google App Engine with custom & flexi environment using Docker image.

#Google App Engine

#Details
app.yaml - Configuration file for App Engine. This just declares the runtime is custom and to use the Dockerfile to run the application.
Dockerfile - Defines your docker image. It extends from the official nginx Docker image and adds the configuration and static files.
apache2.conf - A basic custom apache2 configuration file.
web - Drupal web Root dir to be served by apache using docker on GAE.
php.ini - this file is just a custom inin file if you want to modify something in it.

CMD to deploy on GAE -  got the root path of the application and run below cmd .

gcloud app deploy

