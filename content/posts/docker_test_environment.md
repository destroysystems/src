---
title: "Disposable Test Environment with Docker"
date: 2020-02-22T20:19:00+01:00
draft: false
authors: ['mauricio']
tags: ['docker', 'docker-compose', 'automation']
---
One of my favorite hobbies (or least effective skills) is setting up my dotfiles. I was never able to have a finished version, or even a MVP. I don't know why, but there is something incredibly hard on creating a very basic .vimrc and pushing to a repository. But ok, I digress.

My last approach on it is with Ansible, an awesome tool to configure remote environments in an easy and automated way. It's incredibly powerful and requires almost no setup on the target. More details on that in a future post (maybe?)

So, everytime that I try and work on it again, I want a clean environment to test. I used Virtualbox in the past, creating snapshots and just going back when I wanted to start over. It works, but is heavy and slow. I also tried with Vagrant. I believe that it may work, but I'm not that patient. If someone knows another way to make it quick and easy, lmk. But, meanwhile, there must be something better.

## Well, why not containers?

Yeah, why not? That's actually what we're going to do. I know that this is not the proper use of containers, but they're mine and I do what I want with them. I'll do the following:

- Create a very basic alpine-based container
- Allow SSH access to it
- ???
- Profit!

Yeah, that's all. Simple like that. 

My biggest struggle was having `sshd` running in my host, which made things quite complicated, as I couldn't simply `-p 22:22` my way into the container. Solution: kill that `sshd` dude. 

So, my `Dockerfile` looks like this:

```
FROM alpine
RUN adduser -D alpine
RUN echo "alpine:password" | chpasswd
RUN apk add openrc openssh
RUN openrc
RUN touch /run/openrc/softlevel
ENTRYPOINT rc-service sshd start && /bin/ash
```

Let's divide it in groups and analyze.

```
FROM alpine
RUN adduser -D alpine
RUN echo "alpine:password" | chpasswd
```
So, quite simple. I just start with a plain Alpine image and run two commands to create the account which I'll be using to access the container. Notice that the password is in plain text. I don't care that much because it's just a test container and will run locally, but never (and I said **NEVER**) do such thing in an exposed Dockerfile unless you have a proper way to manage those secrets (env vars, for exeample).

```
RUN apk add openrc openssh
RUN openrc
RUN touch /run/openrc/softlevel
ENTRYPOINT rc-service sshd start && /bin/ash
```

Here we installed the required packages. `openrc` is a service manager, such as `systemctl` or similar tools. The lightweight alpine image doesn't have it. `openssh` is the ssh server and other tools. Maybe we could go with another version, to have only the server and make the container even smaller.

Then we start `openrc` to manage the services running in the container. The next line is required by `openrc` to manage services. More on that in the future. 

And finally, the entry point for the container starts `sshd` and starts the shell. The shell may not be needed for what this container is intended, but I left it in there if any debugging is needed.

## Run that thing

Okay, our `Dockerfile` is created. Let's run it and see what happens. I'll assume that you have docker installed and know the basics. I'm far from being an expert on it, so I'll leave the googling on this part to you.

To build the image for our container we do the following:

```
docker build -t "my_image" .
```

This creates an image based on the Dockerfile that we wrote. Don't forget to run it in the same folder as the Dockerfile, or change `.` in the end of the command to the actual folder that contains the Dockerfile.

After the build finished, let's start a container with that image:

```
docker run -dt -p 22:22 --rm --name my_container my_image
```

This command will do the following:

`docker run`: Starts the container

`-dt`: Runs it detached and in background

`-p 22:22`: Forwards port 22 in the host (before the `:`) to the same 22 in the container (after the `:`)

`--rm`: Deletes the container after it finished running

`--name my_container`: Gives a name to the container. Makes it easier to find later, but isn't mandatory

`my_image`: The image that we just created

This should start your container! Now, to connect to it, just run the normal ssh:

```
ssh alpine@localhost
```

Where `alpine` is the user that we created in the Dockerfile. Change it accordingly. And simply `localhost` because we're forwarding requests from port 22 on localhost to port 22 in the container. This is why I mentioned in the beginning to not have anything running in port 22. Password is `password`

If you have, for example, sshd running in your host, it will be listening in port 22 and you won't be able to use it this way. It's possible to SSH in a different port with the flag `-p`. In this case, the `docker run` command also needs to reflect such changes:

```
docker run -dt -p 6543:22 --rm --name my_container my_image
ssh alpine@localhost -p 6543
```

In my case I don't want to deal with changing the port that Ansible will use, so I went through the trouble of freeing port 22.

And that's it! Now you have a container to which you can quickly SSH and, after your things are done, you just stop it and it's gone!
