---
id: lab4
title: Lab 4 - Containers
sidebar_position: 4
description: Working with containers using Docker in Linux and Windows
---

# Lab 4 - Containers

## Overview

This week's lab will cover the following:

- Pulling and running pre-existing containers
- Starting, stopping, and organizing containers and their images
- Creating and modifying our own containers
- Running a container across multiple operating systems

## Lab 4 Notes

We are going to look at a few examples of containers and how to use them in this lab and we are going to use Docker to do so. However, Docker is not the only way to use containers. Docker is used here because it does a good job of showing how simple containers can be used, particularly between Linux and Windows systems. As you move onto more complex uses of containers in industry, you may find that other container software may be more useful for your specific tasks.

Also, we will be downloading VSCode onto our Mint VM in this lab to help with the creation/modification of container config files. VSCode is not necessary to do this (you could just use VIM or NANO) but it makes the viewing and editing of configuration files much easier and nicer to look at. Just keep in mind that you need a GUI to use it so it won't always be available to you.

Finally, we will show you how to remove/delete containers and their images in this lab. If at any point you get lost with this lab, remember that you can simply delete all your containers and images and start from scratch.

## Before You Get Started...

Before you begin this lab, power on all your VM and double check that the configurations from Lab 3 are still working. You should be able to: 
- Ping win-client.cnet2.ops345.org from mint-client
- Ping mint-client.cnet1.ops345.org from win-client
- Ping www.google.com and/or www.ign.com from all VMs
- Use Samba to share files between your mint-client and your win-client

If all that is correct you can proceed with Lab 4. Otherwise, please double check your configurations from lab 3. 

## Investigation 1: Installing Docker

First we are going to install docker on our mint-client VM. 

Open a terminal on mint-client and run: 
```bash
sudo apt install docker.io
sudo usermod -aG docker hheim
```
**-- Replace “hheim” with your own username --** 

Reboot your mint-client.  

Next we will install it on our win-client VM. Boot up your win-client and log in.

The first thing we are going to do is install WSL (Windows Subsystem for Linux). This is a very powerful tool that allows you to run a virtualized instance of Linux inside Windows. It has many uses but in our case, it is used by Docker to run containers. We will be using an older (and stable) version of WSL as newer versions are known to cause issues.

Open your web browser and go to:
[WSL Releases](https://github.com/microsoft/WSL/releases)

- Scroll down to and click on "2.6.3"
- Now click on the "wsl.2.6.3.0.x64.msi" file. This will download the file to your win-client.
- When the file has finished downloading, double click on it to install WSL.
- When the installation is complete, reboot win-client.

When it comes back up go to the following website: 

[Windows Docker Installer](https://docs.docker.com/desktop/setup/install/windows-install/)

**Note** - if this is your first time opening the Edge web browser you will have to click a few buttons to get going. Just say “No” to everything. 

- Click on the “Docker Desktop for Windows – x86_64” button to download Docker Desktop.
- Once the download finishes, go into your Downloads directory and double click the installer to get it started.
  - The install process may be laggy and slow (because we didn’t give Windows as much RAM/CPU as it wants) but be patient and don’t cancel the install in progress.
- Make sure the “Use WSL 2 instead of Hyper-V” button is checked and click “OK”
- When the installation is complete, click the “Close and restart” button and let Windows reboot.
- When Windows reboots, log in and wait for the Docker service agreement to pop up (this may take a few minutes). Click “Accept”
- A Docker Desktop window should then appear. Click “Skip” and wait for the Docker Engine to start up (this may take a few minutes)
- Once it finishes, you can minimize the Docker Desktop window and your win-client VM for now. 

## Investigation 2: Pulling, Starting, and Stopping Containers 

Now that we have Docker installed, we can take a look at a few simple examples of containers. First, we will pull an existing container image from the Internet. 

In mint-client, open a terminal (you will want to maximize the window for easier reading) and run: 
```bash
docker run hello-world 
```
Notice the output. When you first ran this command, Docker looked for a container called “hello-world” locally but did not find one. So, it automatically went online and pulled the container from a known location. Once it had the container, it ran it successfully, giving you the “Hello from Docker!” message. 

Try running it again: 
```bash
docker run hello-world 
```
This time, the output should start with the “Hello from Docker!” message. Docker did not need to pull the container from the Internet as it has now stored it on your mint-client VM. It can simply run it from that local location. This image will be stored there until it is removed. 

Now try running the following two commands (they effectively do the same thing): 
```bash
docker ps –a 
docker container ls –a 
```
These commands list all the containers that exist on your system. The “-a” options ensures that containers that are not currently running are shown. If you run these commands without the “-a” right now, nothing will appear because no containers are currently running.  

Also, note the container IDs. They are different even though the same “hello-world” image was used. There is a difference between a container and the image it runs. When we ran the “docker run” commands, docker created a container that had everything needed to run the “hello-world” image. A new container is created each time you “run” an image. 

Run this command one more time: 
```bash
docker run hello-world 
```
And now run: 
```bash
docker ps –a 
```
There should now be 3 “hello-world” entries, each with their own container ID and name. 

We can also list just the images we have saved on our system instead of the containers with: 
```bash
docker image ls 
```
This will show you we only have one image right now – the “hello-world:latest” image. 

We won’t be needing 3 different hello world containers so lets remove two of them. To do so, run the following for two of your hello-world containers: 
**(Replace “containerid” with your actual container id or “containername” with your actual container name)**
```bash
docker rm containerid 
```
or 
```bash
docker rm containername 
```

The examples we just used were examples of containers that ran and then stopped. Because the point of the container was to output text, it’s not something that we need to manually start and stop. Let’s try an example of something we can control. 

We will pull a container called “busybox”, a simple container that allows for the running of Linux commands (quite useful when testing simple Linux functions in a non-Linux OS). We are just going to use it to see what a background running container looks like. Run the following: 
```bash
docker pull busybox 
docker image ls 
```
You should now see the “busybox:latest” image has been added to your local image list. Now run: 
```bash
docker run -d --name mybusyboxloop busybox sh -c “while true; do echo running; sleep 5; done” 
```
What we have just done is started a container in which a simple BASH loop will run forever until we stop it. 

Run the following: 
```bash
docker ps -a 
docker ps 
```
Notice that the first command lists both your “hello-world" and  “mybusyboxloop” containers while the second only lists your “mybusyboxloop” container because it is still running. Also, note that the status of "mybusyboxloop" is set to "Up".

Let’s stop that container: 
```bash
docker stop mybusyboxloop 
```
The command may take a moment to process. Once it does, run: 
```bash
docker ps –a 
```
Notice the status of the “mybusyboxloop” container has changed from “Up” to “Exited”. 

Now start the container up again with: 
```bash
docker start mybusyboxloop 
```
Now run the command to list all of your containers and you will see it has changed back to “Up”. Finally, stop it again before moving to the next section. 

## Investigation 3: Building, Running, and Managing our own Containers 

Next, we are going to try building our own containers but before we do that, let’s download VSCode to make our lives a little easier. 

Continue working in your mint-client VM. 

Go to: 
[VSCode Installer](https://code.visualstudio.com/download) 
and click the “.deb Debian, Ubuntu” button. 

The VSCode installer will download. 

Click the installer once it has finished and then click “Install Package”. VSCode will now install. Click “Next” on any configuration screens that appear. Once the installation is complete, click on the “LM” symbol in the bottom left hand side of the screen and type “vscode”. Click on the VSCode application.  

VSCode is a powerful and useful tool for all sorts of coding, scripting, and configuring. We are going to use it to create two containers. The first will combine some simple BASH script with a Docker instruction file to do a quick IP connection test. 

You can close the default “Walkthrough” tab that starts up. 

To begin, click “View >> Terminal”. This will open a terminal within VSCode. 

In that terminal, navigate to your home directory and create a new directory called “Containers”. Inside of your “Containers” directory, create a new directory called “Pingtest”. Enter that directory. 

In that directy create two empty files: 
```bash
touch ping.sh 
touch Dockerfile 
```
Now open both those files in VSCode (File >> Open File). 

You should get a pop-up when you open the “Dockerfile” asking to install “Container Tools”. Click “Install”. 

Once it installs, enter the following in “ping.sh”: 
```bash
#!/bin/sh 

echo "---------------------------------------" 
echo " Simple Ping Tool (inside a container) " 
echo "---------------------------------------" 

read –p “Enter IP or FQDN to ping: “ TARGET 

# test to see if variable is empty 

if [ -z "$TARGET" ]; then 
  echo "No target provided. Exiting." 
  exit 1 
fi 

# try 4 pings with a 2 second timeout per packet, then provide result to user

if ping -c 4 -W 2 "$TARGET" >/dev/null 2>&1; then 
  echo "Ping to \"$TARGET\" was successful." 
  exit 2
else 
  echo "Ping to \"$TARGET\" failed. Destination unreachable." 
  exit 3 
fi 
```
Save your file and then give is execute permissions:  
```bash
chmod +x ping.sh 
```

Enter the following in “Dockerfile”: 
```bash
# Pull from the alpine repo 
FROM alpine:3.19 

# Install iputils 
RUN apk add --no-cache iputils 

WORKDIR /app 

COPY ping.sh /usr/local/bin/ping.sh 

RUN chmod +x /usr/local/bin/ping.sh 

# Default:start the interactive prompt 

CMD ["/usr/local/bin/ping.sh"] 

```
Save your file. 

Now make sure you are in the “Pingtest” directory and run the following: 
```bash
docker build –t ping-test . 
```
This will build your docker image using the instructions in the docker file you just created.   

Now run the image: 
```bash
docker run --name ping-test --rm –it ping-test 
```
(be careful with the command above, make sure you are using the correct amount of dashes) 

Don’t immediately put in an IP or FQDN to test. First, open a second terminal window and run the command to get a full listing of all your containers. Notice that your “ping-test” container is listed as “Up” in the output.  

Go back to your other window and enter an IP or FQDN for your container to try to ping win-client.  

Whatever the result, go back to your second terminal window and run container listing command again. Your container is not listed as “Exited”. Your container is not there at all. Why? 

Let’s break down our “docker run” command: 

- The command begins with “docker run” to run a specified image in a container.
- We use the “--name” flag and “ping-test” to name our container the same name as the image we are using (the one we just built).
- The “--rm” flag removes the container once we are done with it.
- The “-it” flag allows the user to input data into the script in the container as it is running.
- Finally, “ping-test” is the name of the image we are running in the container. 

So because of the “--rm’ flag, our container is automatically deleted once it has finished running. This is useful to keep in mind so we don’t clutter up our containers list.  

Run the command one more time without the “--rm” flag: 
```bash
docker run --name ping-test –it ping-test 
```
The container should run as expected. Now check your list of containers in the other terminal window: 
```bash
docker ps –a 
```
Without the “--rm” flag, the container is still there. Try running the image again, without specifying a name or the “-it” flag: 
```bash
docker run ping-test 
```
Notice the output. The container didn’t allow us to enter anything so the “$TARGET” variable in our script remained empty and the script exited without trying to ping anything. This is because we didn’t add the “-it” flag to our “docker run” command. Try adding in that flag: 
```bash
docker run -it ping-test 
```
The container should run correctly now but take a look at your containers list: 
```bash
docker ps –a 
```
Notice that our “ping-test” image has been used 3 times in 3 different containers. We have our “ping-test” container but we also have 2 randomly named containers because we didn’t specify names for them. This is because we used “docker run” and not “docker start”. We were creating new containers each time instead of starting a pre-existing one.  

Remove the two randomly named ping-test containers: 
**(replace the "randomcontainernames" with your acual container names)**
```bash
docker rm randomcontainername1
docker rm randomcontainername2 
```

Now check your container images: 
```bash
docker image ls -a 
```
Our ping-test image is still there. This allows us to use it whenever we want. Keep this in mind. The containers themselves are disposable. It is the images that we generally need. But even if we were to delete those, we could always rebuild them with the files we created. Let’s do that now with our ping-test image. 

First, delete the last ping-test container: 
```bash
docker rm ping-test 
```
Confirm the container was removed: 
```bash
docker ps –a 
```
Now delete the ping-test image: 
```bash
docker rmi ping-test:latest 
```
Confirm the image was removed: 
```bash
docker image ls -a  
```
Now that it is gone, let’s rebuild that image. Make sure you are in the Pingtest directory and run: 
```bash
docker build –t ping-test . 
```
Confirm the image was rebuilt: 
```bash
docker image ls -a 
```
You should see the image is back. You will also notice a few “untagged” images. These are “intermediate layer” images. They are basically different functions used to make the main image work. In this case they are each attached to a line in the “Dockerfile” (WORKDIR, COPY, RUN etc). These “untagged” images are like hidden files. They don’t show up without the -a option. 

Now run the image just like we did the first time but without deleting it afterward: 
```bash
docker run --name ping-test -it ping-test 
```
After it has run, check to see that the new container is on our list: 
```bash
docker ps -a 
```
You should see the ping-test container which is using the new ping-test image. 

This time, we will use the existing ping-test container instead of creating a brand new one: 
```bash
docker start ping-test 
```
What happened? Why did we not get asked to input an IP or FQDN for our container to test? 

Our container started successfully. You can see this when you look at the running containers: 
```bash
docker ps –a 
```
You will see that the ping-test container is “Up”. It will remain “Up” forever because it is awaiting input and we have no way of providing that. We need a flag in our “docker start” command to allow that. 

Let’s stop the currently running ping-test container: 
```bash
docker stop ping-test 
```
This time we will start it with the correct flag: 
```bash
docker start -i ping-test  
```
You should now be able to input the required information for the script inside the container to run properly and the container will stop when it has finished.  

Now let’s create another container that does something different. We are going to create a container that launches a very simple version of an Apache web server. We will talk more about Apache specifically in the second half of this course but for now we are just going to use it as another example of a perpetual container that runs until we stop it. 

Begin by going into your “Containers” directory and creating a new directory called “Apache”. Inside of that directory, create the following two empty files: 
```bash
touch index.html 
touch Dockerfile 
```
Using VSCode enter the following into index.html: 

```bash
<!DOCTYPE html> 
<html> 
	<head> 
		<meta charset="UTF-8"> 
		<meta name="viewport" content="width=device-width, initial-scale=1.0"> 
		<meta name="author" content="Hans Heim"> 
		<title>Hans’ Web Page</title> 
	</head> 
	<body> 
		<h1>Hans's web page</h1> 
		<p>Hello World</p> 
		</body> 
</html> 
```
Change the “author” line to your name, the title line to your own tile, and the header1 line to your name. Save the file. 

Enter the following into Dockerfile: 
```bash
FROM httpd:latest 
WORKDIR /usr/local/apache2/htdocs 
COPY . /usr/local/apache2/htdocs 
EXPOSE 80 
```
Save the file and return to your terminal window. Make sure you are in the "Apache" directory and run the following: 
```bash
docker build -t my-apache-app . 
```
This will create your “my-apache-app" image. Check to see that it has been added to your local images list: 
```bash
docker image ls 
```
Now run the image: 
```bash
docker run --name my-apache-app -d -p 80:80 my-apache-app 
```
Note that we are providing a few new options here: 
- The “-d” option allows us to run this container in the background
- The “-p” 80:80 forces the ports this container will use 

Now check to see that your container is running with: 
```bash
docker ps –a 
```
It should be “Up”. 

Next, open your web browser and enter the following into the address bar: 

http://localhost (Be sure that it is http and not https) 

You should see your basic web page appear. Close your web browser. 

Boot up your win-client VM, log in, and open up the web browser. Type the following into the address bar: 

http://mint-client.cnet1.ops345.org 

You should see the webserver that is being hosted inside a container on your mint-client machine. Pretty cool! 

Back in your mint-client, stop your my-apache-app container: 
```bash
docker stop my-apache-app 
```
On your win-client, open a new web browser tab and try to go to the webserver again. It should fail because you stopped the container (you may need to refresh the webpage). However, we did not delete the container. It is still there ready for use. All we have to do is start it up again. Do that now: 
```bash
docker start my-apache-app 
```
Now, once more, try accessing the webpage in your win-client. The web page should appear successfully once again. 

Before moving forward, stop your “my-apache-app" container once more.

## Investigation 4: Using Containers Across Operating Systems 

Up until now, all the containers that we have used have been run out of our mint-client VM. But one of the most useful aspects of containers is that they can be ported to different systems without fear of missing any dependencies or underlying technology. The containers carry all of that with them so (generally speaking) we can transfer containers between systems and they should work the same as on the system they were built. 

First, let’s pull a pre-existing container and see how it runs on our mint-client and how it runs on our win-client. 

On your mint-client run: 
```bash
docker run –d --name pong –p 8080:80 danielou1/ping-pong-game:latest 
```
Then, open your web browser and put the following into the address bar: 

http://localhost:8080 

You should be able to play a little pong. I dare all of you to score even a single point (But try not to spend the rest of your class time playing). 

Now, let’s try that on our win-client.  Log into your win-client and open Docker Desktop. Wait for the application and the Docker Engine to start up. Once it is running, open Powershell as an Administrator and input the following command to make sure Docker is running correctly: 
```bash
docker ps –a 
```
You won’t get any containers as you haven’t started any yet but you should get the same column headers you got when running this command on mint-client (CONTAINER ID, IMAGE, COMMAND etc.) 

Now try running the same command we just ran on mint-client: 
```bash
docker run –d --name pong –p 8080:80 danielou1/ping-pong-game:latest 
```
The command will probably take a bit longer to process than it did on mint-client so be patient. Once it finishes, open your web browser and try accessing the game just like you did on mint-client. Again, try not to get too distracted by the absolute intensity that is Pong. 

We can run the same commands in Powershell that we did in mint-client to check the status of our containers: 
```bash
docker ps –a 
```
But before we stop our pong container, let’s quickly take a look at the Docker Desktop application.  

Opening it up and navigating to the “Containers” heading on the left hand side will show that we have a single container called “pong”. Clicking on that container allows us to view information about the container and its current status. 

We can also click on “Images” on the left hand side to see the images that we can spin up inside containers.  

Docker Desktop is a useful tool that can be used on most platforms (including our mint-client VM if we wanted to). However, keep in mind that there are many scenarios where we will not have a GUI desktop and will therefore be limited to using the command line. 

Go back into your Powershell terminal and stop your pong container: 
```bash
docker stop pong 
```
Confirm in both Powershell and Docker Dekstop that the pong container has “Exited”. 

The last thing we are going to do is create a brand new container and manually move it over to another system. 

On your mint-client VM, navigate to your “Containers” directory and create a new directory called “Clickit”. Enter into the new directory and create an empty file: 
```bash
touch Dockerfile 
```
Create a new directory inside your “Clickit” directory called “game” and inside of that directory create an empty file: 
```bash
touch index.html 
```
Open VSCode and open both of these files. 

Enter the following in index.html (Make sure to type it all out by hand)(Just kidding, you can copy/paste):

```bash
<!doctype html> 
<html lang="en"> 
<head> 
  <meta charset="utf-8" /> 
  <meta name="viewport" content="width=device-width,initial-scale=1" /> 
  <title>Click the Box</title> 
  <style> 
    :root { color-scheme: dark; } 
    body { 
      margin: 0; height: 100vh; display: grid; place-items: center; 
      background: #0f172a; color: #e2e8f0; font-family: system-ui, Arial, sans-serif; 
    } 
    .hud { position: fixed; top: 12px; left: 12px; right: 12px; display: flex; justify-content: space-between; font-weight: 600; } 
    #board { position: relative; width: min(90vw, 700px); height: min(70vh, 500px); background: #111827; border: 2px solid #334155; border-radius: 10px; } 
    .box { 
      position: absolute; width: 60px; height: 60px; border-radius: 8px; 
      display: grid; place-items: center; cursor: pointer; user-select: none; 
      box-shadow: 0 6px 18px rgba(0,0,0,.35); transition: transform .05s ease; 
      background: linear-gradient(135deg, #22c55e, #16a34a); 
    } 
    .box:active { transform: scale(0.95); } 
    button { padding: .5rem .75rem; background: #2563eb; border: none; color: white; border-radius: 6px; cursor: pointer; } 
    button:hover { background: #1d4ed8; } 
  </style> 
</head> 
<body> 
  <div class="hud"> 
    <div>🎯 Score: <span id="score">0</span></div> 
    <div>⏱️ Time: <span id="time">30.0</span>s</div> 
    <div><button id="restart">Restart</button></div> 
  </div> 
  <div id="board"></div>

  <script> 
    const board = document.getElementById('board'); 
    const scoreEl = document.getElementById('score'); 
    const timeEl = document.getElementById('time'); 
    const restartBtn = document.getElementById('restart'); 
 
    let score = 0, timeLeft = 30.0, running = false, timerId = null, box = null; 

    function rand(min, max) { return Math.random() * (max - min) + min; } 
    function clamp(v, min, max) { return Math.max(min, Math.min(max, v)); } 

    function spawnBox() { 
      if (box) box.remove(); 
      box = document.createElement('div'); 
      box.className = 'box'; 
      const bRect = board.getBoundingClientRect(); 
      const size = 60; 
      const x = rand(0, bRect.width - size); 
      const y = rand(0, bRect.height - size); 
      box.style.left = `${x}px`; 
      box.style.top = `${y}px`; 
      box.textContent = '+1'; 
      box.onclick = () => { 
        if (!running) return; 
        score++; 
        scoreEl.textContent = score; 
        spawnBox(); 
      }; 
      board.appendChild(box); 
    } 

    function start() { 
      score = 0; timeLeft = 30.0; running = true; 
      scoreEl.textContent = score; timeEl.textContent = timeLeft.toFixed(1); 
      spawnBox(); 
      if (timerId) cancelAnimationFrame(timerId); 
      let last = performance.now(); 
      const tick = (now) => { 
        const dt = (now - last) / 1000; last = now; 
        if (!running) return; 
        timeLeft = clamp(timeLeft - dt, 0, 999); 
        timeEl.textContent = timeLeft.toFixed(1); 
        if (timeLeft <= 0) { 
          running = false; 
          if (box) box.textContent = '🏁'; 
          alert(`Time! Your score: ${score}`); 
          return; 
        } 
        timerId = requestAnimationFrame(tick); 
      }; 
      timerId = requestAnimationFrame(tick); 
    } 

    restartBtn.addEventListener('click', start); 
    window.addEventListener('resize', () => { if (running) spawnBox(); }); 

    // auto-start 
    start(); 
  </script> 
</body> 
</html> 
```
Save your file. 

Enter the following in Dockerfile: 
```bash
# Pull the nginx web server 
FROM nginx:alpine 

# Copy the game files to the  nginx web root 
COPY . /usr/share/nginx/html 

# Expose HTTP port 
EXPOSE 80 
```
Save your file. 

Enter your “Clickit” directory and run the following command:
```bash
docker build –t clickit-game . 
```

Confirm your “clickit” image has been built: 
```bash
docker image ls 
```
And then run your new image: 
```bash
docker run –d --name clickit-game  –p 8080:80 clickit-game 
```
You should be met with an error saying a container network driver failed to set up. This is because our old pong container is still running and using the port we want to use. We will have to stop the pong container before we can properly set up a container for our clickit game. 

Stop the pong container: 
```bash
docker stop pong 
```
We will also remove the new clickit container that we just created as it wasn’t created properly (we could just create a new one with a different name but to keep things organized we will delete the old one and start again): 
```bash
docker rm clickit-game 
```
Now try running the clickit image one more time: 
```bash
docker run –d --name clickit-game  –p 8080:80 clickit-game 
```
Go back into your web browser and put the following into the address bar: 

http://localhost:8080 

When you hit enter, you may see the old pong game load up again because it is stored in your browser’s cookies but refreshing the page will start up the clickit game. See if you can beat my high score of 50. Bet you can’t. 

Close your browser and go back to your terminal and list your containers with: 
```bash
docker ps –a 
```
You should see that your clickit container is still running. Stop it: 
```bash
docker stop clickit-game 
```
We are now going to prepare our clickit container and send it over to our win-client to demonstrate that the container has everything the game needs to run, even on a totally different operating system. 

Start up your win-client VM if it isn’t already on and start up Docker Desktop. Also, double check that you can still access the Samba share on it. If the share is not currently running, you can re-add it using the steps from the previous lab. 

On your mint-client VM, navigate to your “sambadir” directory and run: 
```bash
docker save –o clickit-game.tar clickit-game 
```
Check to make sure that the “clickit-game.tar” file is now located in your “sambadir” directory. 

Switch over to your win-client VM. In win-client, create a directory called “Clickit” in the root of your C:\ drive. Then copy the “clickit-game.tar” file from the share directory into the “Clickit” directory. 

Run Powershell as an administrator and navigate to your “Clickit” directory. Then run: 
```bash
docker load –i .\clickit-game.tar 
```
You should receive a “Loaded image” message. Now run: 
```bash
docker image ls 
```
Your clickit-game image should now appear, letting you know the image is now stored locally. You can also click on “images” in the Docker Desktop application to see the image has been successfully added. 

Finally, run the image: 
```bash
docker run –d --name clickit-game  –p 8080:80 clickit-game 
```
Open your web browser and go to http://localhost:8080 and you should be able to play Clickit on your win-client. (Again, if you see Pong when you first go to the website, refresh it and Clickit should appear). 

You have now successfully transferred your Clickit game over to Windows! Don’t forget to stop the clickit-game container when you’re done. 

## Lab 4 Sign-Off

Take screenshots showing the following: 

- a listing of all containers on your mint-client
- a listing of all container images on your mint-client
- the clickit game successfully running in your mint-client
- the clickit game successfully running in your win-client

## Exploration Questions

