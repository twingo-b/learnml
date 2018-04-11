---
title: Using Azure
---

# Using Azure

For seminars with participants that have different personal computing setups, it can be useful to take advantage of cloud-based solutions in order to ensure everyone has access to the same hardware and software. Countless options exist, and while I have no particular preferences for any one service over another, in February 2018 I carried out a series of lectures using Microsoft's Azure service, in which 25+ participants were assigned their own instance of an Ubuntu-based *Data Science Virtual Machine* (DSVM). On this page, a few notes are recorded for posterity.

## Jupyter notebooks run remotely, viewed locally

Most of the information online regarding use of Jupyter notebooks on Azure seems to suggest using the X2Go Client to provide GUI access to the remote machine, within which Firefox (the default browser) can be used to view and edit Jupyter notebooks. This introduces some serious latency to each step in one's workflow. If all operations are going to be done via in-browser notebooks, then a better solution is to simply use SSH to log in to the remote machine, and using port forwarding, access the Jupyter server via a browser on the *local* machine.

### Windows case

Let's go through an example on Windows 10 (analogous procedures can be done with ssh from the shell of a UNIX-like system). To gain access to the remote shell, we use <a href="https://www.putty.org/">PuTTY</a>. When setting up the Azure DSVM (among other setups) there are two choices: (1) a password-based login, or (2) a SSH-based login. In the latter case, a public key of the proper format must be available on the local machine. The former requires just a user name and password at login. The latter requires in addition to this a public key on the local machine in the correct form. In any case, let's say we'll use port 8888 locally, and port 8889 remotely (these are typical, but arbitrary choices). The former is the source, and the latter is the destination. Setting up port forwarding in PuTTY:

<img src="img/portforward_putty.png" alt="IMG: Port forwarding using Putty" />

With this setup, initiate the connection by giving PuTTY the remote machine's public IP address and the user name (and public key, if applicable), and log in, presenting the password when prompted.

Assuming a successful login, for our example we move to our desired working directory, and start up Jupyter notebook without a browser, using the desired port:

```
$ cd learnml
$ jupyter notebook --no-browser --port=8889
```

After executing this command, a series of messages proceeds in the terminal window, culminating in the following:

```
The Jupyter Notebook is running at: http://localhost:8889/?token=XXXXXXXXXXX
```

where `XXXXXXXXXXX` is some sequence of alpha-numeric characters.

Now, since we have set up our port forwarding from 8888 (local) to 8889 (remote), in terms of local operations, all we need to do is open up our local browswer, and in the URL address bar enter

```
http://localhost:8888/?token=XXXXXXXXXXX
```

noting that everything in this URL is the same as in the remote terminal, except we have replaced `8889` with `8888`. If all has worked correctly, we should have access to the remote Jupyter notebooks, with the functional convenience of a local browser.

### Ubuntu case (16.04 LTS)

The UNIX-like case is even simpler than the Windows case. Here we use `ssh` from the command line to do SSH port-forwarding.

On the remote machine, once again, run

```
$ cd learnml
$ jupyter notebook --no-browser --port=8889
```

to get the notebook server going. Then on the local machine, start the tunnel:

```
$ ssh -N -L localhost:8888:localhost:8889 REMOTE_USER@REMOTE_HOST
```

where `REMOTE_USER` is replaced with your username, and `REMOTE_USER` is the name or IP address of the remote server. Then from the browser of your local machine, access `localhost:8888` using the URL including a token, exactly as shown in the previous example.


### Alternative to tokens: setting up a password-based notebook server

In the above examples, we considered the default case of using randomly-generated tokens to access the notebook server remotely. Doing the copying and pasting described above can be a bit of a nuisance, and indeed depending on your environment, copying from the remote desktop to the local one may require a bit of dexterity. This hassle can be easily circumvented by using a password set in advance for moderating access to the notebook server. To do this is simple. On the remote machine, run the following:

```
$ cd learnml
$ jupyter notebook password
> Enter password:
> Verify password:
```
It should then respond with `Wrote hashed password to ...` where the path to a particular `.json` file is specified.

With this password in place, in the above examples, instead of typing in the long URL+token string of text into the browser, one can simply enter `http://localhost:8888` and provide the password when prompted.