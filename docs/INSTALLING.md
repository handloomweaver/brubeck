# Installing the environment

First, we have to install a few things. Brubeck depends on Mongrel2, ZeroMQ and a few python packages.

All three packages live in github, so we'll clone the repos to our Desktop.

    $ cd ~/Desktop
    $ git clone https://github.com/j2labs/brubeck.git
    $ git clone https://github.com/zedshaw/mongrel2.git
    $ wget http://download.zeromq.org/zeromq-2.1.7.tar.gz 
    $ tar zxf zeromq-2.1.7.tar.gz


## ZeroMQ

For us, ZeroMQ is actually two pieces: libzmq and pyzmq. libzmq must be installed by hand like you see below.

    $ cd ~/Desktop/zeromq-2.1.7    
    $ ./autogen.sh
    $ ./configure  ## for mac ports use: ./configure --prefix=/opt/local
    $ make 
    $ sudo make install


## Mongrel2

Mongrel2 is also painless to setup.

    $ cd ~/Desktop/mongrel2
    $ make  ## for mac ports use: make macports
    $ sudo make install


## Python packages & Brubeck

First, brubeck work great with virtualenv. I recommend using

If you have pip installed, you can use the requirements file. 

    $ cd ~/Desktop/brubeck
    $ pip install -I -r ./requirements.txt

If you don't have `pip`, you can `easy_install` the libraries listed.


### Brubeck itself

As the last step, install Brubeck.

    $ cd ~/Desktop/brubeck
    $ python setup.py install


# A demo

Assuming the environment installation went well we can now turn on Brubeck.

First, we setup the Mongrel2 config.

    $ cd ~/Desktop/brubeck/demo
    $ m2sh load -config mongrel2.conf -db dev.db
    $ m2sh start --db dev.db --host localhost

Now we'll turn on a Brubeck instance.

    $ cd ~/Desktop/brubeck/demos
    $ ./demo_minimal.py

If you see `Brubeck v0.x.x online ]------------` we can try loading a URL in a browser. 
Now try [a web request](http://localhost:6767/brubeck/).


## Mongrel2 configuration

Mongrel2 is a separate process from Brubeck, so it is configured separately.

This is what the Mongrel2 configuration looks like for the demo project.

    brubeck_handler = Handler(
        send_spec='ipc://127.0.0.1:9999',
        send_ident='34f9ceee-cd52-4b7f-b197-88bf2f0ec378',
        recv_spec='ipc://127.0.0.1:9998', 
        recv_ident='')

    brubeck_host = Host(
        name="localhost", 
        routes={'/': brubeck_handler})
    
    brubeck_serv = Server(
        uuid="f400bf85-4538-4f7a-8908-67e313d515c2",
        access_log="/log/mongrel2.access.log",
        error_log="/log/mongrel2.error.log",
        chroot="./",
        default_host="localhost",
        name="brubeck test",
        pid_file="/run/mongrel2.pid",
        port=6767,
        hosts = [brubeck_host])
    
    settings = {"zeromq.threads": 1}
    
    servers = [brubeck_serv]
    
In short, it says any requests for `http://localhost:6767/` should be sent to the Brubeck handler. 

The web server is also configured to answer requests on port `6767`, logging to `./log` directory and puts the mongrel2 pid in a pidfile in the `./run` directory.

# Brubeck.io
