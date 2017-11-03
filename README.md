# dockerWork
Work related to Docker training done following https://docs.docker.com/get-started/


For the part https://docs.docker.com/get-started/part2/#apppy , I typed:
	- create a file called Dockerfile: it contains the container's definition.

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


For https://docs.docker.com/get-started/part3/#introduction (we take the app we wrote in part 2, and define how it should run in production by turning it into a service, scaling it up 5x in the process), I typed:
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


For https://docs.docker.com/get-started/part4/ (we deploy the application onto a cluster, running it on multiple machines. Multi-container, multi-machine applications are made possible by joining multiple machines into a “Dockerized” cluster called a swarm.), I typed:
	- Understanding Swarm clusters:
		- A swarm is a group of machines that are running Docker and joined into a cluster. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a swarm manager. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as nodes.
		- Swarm managers can use several strategies to run containers, such as “emptiest node” – which fills the least utilized machines with containers. Or “global”, which ensures that each machine gets exactly one instance of the specified container. You instruct the swarm manager to use these strategies in the Compose file, just like the one you have already been using at Part 3.
		- Swarm managers are the only machines in a swarm that can execute your commands, or authorize other machines to join the swarm as workers. Workers are just there to provide capacity and do not have the authority to tell any other machine what it can and cannot do.
		- Up until now, you have been using Docker in a single-host mode on your local machine. But Docker also can be switched into swarm mode, and that’s what enables the use of swarms. Enabling swarm mode instantly makes the current machine a swarm manager. From then on, Docker will run the commands you execute on the swarm you’re managing, rather than just on the current machine.
	- Set up your swarm:
		- to enable swarm mode and make your current machine a swarm manager: docker swarm init
		- to have other machines join the swarm as workers, run on them: docker swarm join
		- to create 2 VMs using docker-machine and the Oracle VirtualBox driver:
				docker-machine create --driver virtualbox myvm1
				docker-machine create --driver virtualbox myvm2
		- list the 2 machines and get their IP addresses: docker-machine ls
				NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
				myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce   
				myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce   
		- instruct myvm1 to become a swarm manager: docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.99.100"
				Swarm initialized: current node (y60pkylfrze39ugwful54gczs) is now a manager.
				To add a worker to this swarm, run the following command:
					docker swarm join --token SWMTKN-1-0k6357zjmm32gyd06khvfkyp6mli5vd0b7muq5w9dhg1mip9qc-5jupltqte3i7gmf5tloaupjf8 192.168.99.100:2377
				To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
		- important note:
				- Always run docker swarm init and docker swarm join with port 2377 (the swarm management port), or no port at all and let it take the default.
				- The machine IP addresses returned by docker-machine ls include port 2376, which is the Docker daemon port. Do not use this port or you may experience errors.
		- instruct myvm2 join your new swarm as a worker: docker-machine ssh myvm2 "docker swarm join --token SWMTKN-1-0k6357zjmm32gyd06khvfkyp6mli5vd0b7muq5w9dhg1mip9qc-5jupltqte3i7gmf5tloaupjf8 192.168.99.100:2377"
				- This node joined a swarm as a worker.
		- to view the nodes in this swarm: docker-machine ssh myvm1 "docker node ls"
			ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
			y60pkylfrze39ugwful54gczs *   myvm1               Ready               Active              Leader
			kthq43fweqqsg8y9x603aoxve     myvm2               Ready               Active              
		- to leave a swarm:
				- If you want to start over, you can run docker swarm leave from each node.
	- Deploy your app on the swarm cluster:
		- just repeat the process you used in part 3 to deploy on your new swarm. Just remember that only swarm managers like myvm1 execute Docker commands. Workers are just for capacity.
		- so far, we’ve been wrapping Docker commmands in docker-machine ssh to talk to the VMs. Another option is to run docker-machine env <machine> to get and run a command that configures your current shell to talk to the Docker daemon on the VM. This method works better for the next step because it allows you to use your local docker-compose.yml file to deploy the app “remotely” without having to copy it anywhere.
		- docker-machine env myvm1
				export DOCKER_TLS_VERIFY="1"
				export DOCKER_HOST="tcp://192.168.99.100:2376"
				export DOCKER_CERT_PATH="/Users/philippebrossier/.docker/machine/machines/myvm1"
				export DOCKER_MACHINE_NAME="myvm1"
				# Run this command to configure your shell: 
				# eval $(docker-machine env myvm1)
		- eval $(docker-machine env myvm1)
		- to verify that myvm1 is now the active machine: docker-machine ls
				NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
				myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce   
				myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce 
		- now deploy:
				- note that we are connected to myvm1 by means of the docker-machine shell configuration, and we still have access to the files on your local host. 
					cd /Users/philippebrossier/code_perso/dockerWork
					docker stack deploy -c docker-compose.yml getstartedlab
		- to list the tasks for our service: docker stack ps getstartedlab
				.ID                  NAME                  IMAGE                         NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
				qf2mgaf5zqgb        getstartedlab_web.1   brossierp/get-started:part2   myvm2               Running             Running 2 minutes ago                       
				i9dsr7tnvw3f        getstartedlab_web.2   brossierp/get-started:part2   myvm1               Running             Running 2 minutes ago                       
				qnk63i3ae52w        getstartedlab_web.3   brossierp/get-started:part2   myvm2               Running             Running 2 minutes ago                       
				n0u7ctz5s62j        getstartedlab_web.4   brossierp/get-started:part2   myvm2               Running             Running 2 minutes ago                       
				uazxu3t3ywqw        getstartedlab_web.5   brossierp/get-started:part2   myvm1               Running             Running 2 minutes ago    

				--> services (and associated containers) have been distributed between both myvm1 and myvm2.
		- to copy files across machines: docker-machine scp <file> <machine>:~ 
	- Access your cluster:
		- to get your VMs’ IP addresses: docker-machine ls
					NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
					myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce   
					myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce   
		- to contact app on myvm1: curl http://192.168.99.100:80
		- to contact app on myvm2: curl http://192.168.99.101:80
	- Scale your app:
		- From here, you can do everything you learned about in parts 2 and 3.
		- Scale the app by changing the docker-compose.yml file.
		- Change the app behavior by editing code, then rebuild, and push the new image. (To do this, follow the same steps you took earlier to build the app and publish the image).
		- In either case, simply run docker stack deploy again to deploy these changes.
		- You can join any machine, physical or virtual, to this swarm, using the same docker swarm join command you used on myvm2, and capacity will be added to your cluster. Just run docker stack deploy afterwards, and your app will take advantage of the new resources.
	- Cleanup and reboot
		- tear down the stack: docker stack rm getstartedlab
		- to remove the swarm: we did not run these commands as we need it for tutorial Part 5.
				- for the worker myvm2: docker-machine ssh myvm2 "docker swarm leave"
				- for the manager myvm1: docker-machine ssh myvm1 "docker swarm leave --force"
		- to unset the docker-machine environment variables in your current shell: eval $(docker-machine env -u)	
		- to verify it worked: docker-machine ls
				NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
				myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.10.0-ce   
				myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.10.0-ce   

				--> myvm1 does not have the * at ACTIVE any more.
		- to restart Docker machines:
				- If you shut down your local host, Docker machines will stop running. You can check the status of machines with: docker-machine ls
				- to restart a machine that has stopped: docker-machine start <machine-name>


For https://docs.docker.com/get-started/part5/
	- A stack is a group of inter-related services that share dependencies, and can be orchestrated and scaled together. A single stack is capable of defining and coordinating the functionality of an entire application (though very complex applications may want to use multiple stacks).
	- Some good news: you have technically been working with stacks since part 3, when you created a Compose file and used docker stack deploy. But that was a single service stack running on a single host, which is not usually what takes place in production. Here, you will take what you’ve learned, make multiple services relate to each other, and run them on multiple machines.
	- Add a new service and redeploy:
		- edit docker-compose.yml and add a section for visualizer:
				- The only thing new here is the peer service to web, named visualizer. You’ll see two new things here: a volumes key, giving the visualizer access to the host’s socket file for Docker, and a placement key, ensuring that this service only ever runs on a swarm manager – never a worker. That is because this container, built from an open source project created by Docker, displays Docker services running on a swarm in a diagram.
		- configure your shell to talk to myvm1:
				docker-machine env myvm1
				eval $(docker-machine env myvm1)
				docker-machine list --> make sure you are connected to myvm1 as indicated by an asterisk next to it.
		- re-run the docker stack deploy command on the manager:
				docker stack deploy -c docker-compose.yml getstartedlab
		- 



