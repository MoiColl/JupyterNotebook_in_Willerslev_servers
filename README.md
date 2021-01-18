# Jupyter Notebook in Willerslev servers

###### Moisès Coll Macià
###### 18/01/21


In this tutorial I'm going to explain the solution I found to run jupyter notebook in the Willerslev servers remotely from by personal computer. I'm mainly based on [this](https://medium.com/@sankarshan7/how-to-run-jupyter-notebook-in-server-which-is-at-multi-hop-distance-a02bc8e78314) blog post which explains a similar problem. The different steps are shown in **Figure 1** and explained below. The idea is to create an *ssh tunnel* from a computing node (where your `jupyther notebook` is running) to the front-end of Willerslev servers (Step 1 and 2) and another tunnel from the front-end in Willerslev servers to your working station (Step 3) to open jupyter notebook in your computer browser (Step 4). I assume you've already installed jupyter notebook and all it's dependences and you know how to login to the Willerslev servers. 

![](Figure1.png)

**Figure 1.** Schematic representation to run Jupyter Notebook in Willerslev servers. Highlighted are:
- Yellow : KU username
- Red : computing node name
- Green : port 1
- Purple : port 2
- Cyan : port 3

### Step 1. Running jupyter notebook in a computer node

- Log into the Willerslev servers (on Figure 1.1, you should replace the Xs highlighted in yellow for your user).
- Check which computing node you want to run jupyter notebook on.
- Log into the computing node (on Figure 1.1, represented as Ss highlighted in red)
- Run jupyter notebook in `--no-browser` mode. You also must decide a port, which is a 4 digits number (on Figure 1.1, represented as 9s highlighted in green) which must be unique to your connection, otherwise it gives problems. 


```python
ssh XXXXXX@ssh-snm-willerslev.science.ku.dk
tmon
ssh SSSSSS
jupyter lab --no-browser --port=9999
```

### Step 2. Open the first ssh tunnel

    - On a new terminal, log into the Willerslev servers (on Figure 1.2, you should replace the Xs highlighted in yellow for your user).
    - Create an shh tunnel with the command shown in Figure 1.2. Ss highlighted in red represent the computing node on which you are running the jupyter notebook. Again, you must decide a new port (on Figure 1.2, represented as 8s highlighted in purple) and use the type the port you used before in the jupyter notebook command (on Figure 1.2, represented as 9s highlighted in green)
ssh XXXXXX@ssh-snm-willerslev.science.ku.dk
ssh SSSSSS -L 8888:localhost:9999 -N
### Step 3. Open the second ssh tunnel

    - On a new terminal, create another shh tunnel with the command shown in Figure 1.3. Xs highlighted in yellow represent your user to connect to Willerslev servers. Again, you must decide a new port (on Figure 1.3, represented as 7s highlighted in cyan) and use the type the port you used before in the first tunnel (on Figure 1.3, represented as 8s highlighted in purple)
ssh XXXXXX@ssh-snm-willerslev.science.ku.dk -L 7777:localhost:8888 -N
### Step 4. Jupyter notebook in your local machine browser

    - Open your favourite browser and type `localhost:777`
    - TADAAAA!
    
### Caveats and considerations

1. Difficult to automatize

I found a bit annoying to repeat all steps every time I want to work on jupyter notebook. I added some of the commands in my `.bash_profile` as aliases, but I'm not sure is the best way to automatize the whole thing.

2. Port uniqueness

While port 1, 2 and 3 can be the same 4-digit number, if there are multiple users using the same ports in the same "computer" it's going to create some conflicts and errors. 

3. Close shh tunnels

Sometimes, when I close the shh tunnels (Cntl+c or Cntrl+z), the process keeps running on the background, meaning that the port is still in use. Then, if I try to open again the tunnels, I get the error that the port is on use. To solve that, I kill the process that it's running that particular port with the following command
for job in `ps aux | egrep 9999 | egrep "ssh" | egrep XXXXXX | awk '{print $2}'`; do kill -9 ${job}; done
This selects from all processes running, the ones that have the "9999" (port-id), "ssh" and "XXXXXX" (username) and kill them. 

4. ssh termination

Sometimes, when the ssh doesn't receive orders, it automatically closes down. This kills the ssh tunnel. To prevent that, I first run `screen` so that even when my session is killed, the process goes on and it does not stop my jupyter notebook while working. 

Let me know if you find more problems while using these to run jupyter notebook that are not reported here and if you have improvements and suggestions!