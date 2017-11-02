# dockerWork
Work related to Docker training.


I followed https://docs.docker.com/get-started/


For the part https://docs.docker.com/get-started/part2/#apppy , I typed:
	- to build the app: 
		docker build -t pbtutorialimage .

	- to list my machine’s local Docker image registry:
		docker images
		or
		docker image ls

	- to run the app, mapping my machine’s port 4000 to the container’s published port 80 using -p:
		docker run -p 4000:80 pbtutorialimage
		- the container thinks that Python is serving the app at http://0.0.0.0:80. 
		- the container itself doesn’t know that I mapped its port 80 to my 4000
		- locally, I can use http://localhost:4000 (or curl http://localhost:4000) to see:
				Hello World!
				Hostname: 9a97d86b3262
				Visits: cannot connect to Redis, counter disabled

	- to run the app in the background, in detached mode:
		docker run -d -p 4000:80 pbtutorialimage
		- You get the long container ID for your app and then are kicked back to your terminal. Your container is running in the background. You can also see the abbreviated container ID with: docker container ls

		- to stop the container: docker container stop 1e4064315bf9

	- to share my image:
		- A registry is a collection of repositories.
		- A repository is a collection of images, sort of like a GitHub repository, except the code is already built.
		- An account on a registry can create many repositories. The docker CLI uses Docker’s public registry by default. We’ll be using Docker’s public registry here just because it’s free and pre-configured, but there are many public ones to choose from, and you can even set up your own private registry using Docker Trusted Registry.
		- Log in to the Docker public registry on my local machine: docker login
		- Tag the image: docker tag pbtutorialimage brossierp/get-started:part2
				- pbtutorialimage is the image to tag.
				- brossierp is my username.
				- get-started is the repository name.
				- part2 is the tag name.
		- To see my newly tagged image: docker images
		- To publish the image: docker push brossierp/get-started:part2
				- Once complete, the results of this upload are publicly available. If you log in to Docker Hub, you will see the new image there, with its pull command. See https://cloud.docker.com/swarm/brossierp/repository/docker/brossierp/get-started/general
		- To pull and run the image from the remote repository: docker run -p 4000:80 brossierp/get-started:part2


For https://docs.docker.com/get-started/part3/#introduction , I typed:
	- created a new docker-compose.yml: This is a YAML file that defines how Docker containers should behave in production. Compose files like this are used to define applications with Docker, and can be uploaded to cloud providers using Docker Cloud, or on any hardware or cloud provider you choose with Docker Enterprise Edition.
	- to make our node a swarm manager: docker swarm init
	- to run it with app name set to getstartedlab: docker stack deploy -c docker-compose.yml getstartedlab
	- to get the service ID for the one service in our application: docker service ls
	- to list the tasks for our service: docker service ps getstartedlab_web
		- tasks also show up if you just list all the containers on your system, though that will not be filtered by service: docker container ls -q
	- to test: curl -4 http://localhost
		- you’ll see the container ID change, demonstrating the load-balancing; with each request, one of the 5 tasks is chosen, in a round-robin fashion, to respond. The container IDs will match your output from the previous command (docker container ls -q).
	- to scale up the app:
		- amend the replicas value in docker-compose.yml, save the change, and re-run docker stack deploy -c docker-compose.yml getstartedlab
		- Docker will do an in-place update, no need to tear the stack down first or kill any containers.
	- to take down the app: docker stack rm getstartedlab
	- to take down the swarm: docker swarm leave --force
