Sharing your work with others
=============================

Key-words:
----------

Mac Os X
Vagrant
wordpress
port forwarding
port 80

Introduction
------------

Some framework are capricious and do not accept simple port forwarding. I worked
recently on a wordpress engine, that I needed to access (for responsive design
purposes) with my iPhone and Android phones.

Beyond the need to share the wifi with the mobiles phones (which won't be covered)
you need to forward the port from the vagrant box (port 80) to your host computer.

Since Vagrant runs in user space, it cannot bind/forward on reserved ports (< 1024).
You need to workaround that limitation by forwarding to a port above 1024.

How-to
------

You have to setup your Vagrant file to forward the port 80 of the guest (the
virtual box) to a port of the host (your computer). I choose port 8000 but any
not used over 1024 will do.

	Vagrant.configure("2") do |config|
		config.vm.define :machine do |machine| # allow to access the machine simply
			...
			machine.vm.network "forwarded_port", guest: 80, host: 8000, auto_correct: true
			# guest: 80 means you want vagrant to redirect TO guest port 80
			# host: 8000 means you want to redirect traffic that arrive to your computer port 8000 (to guest port)
			# autocorrect will change 8000 if it is already used on you computer
		end
	end

At this step, you should be able to touch the apache server in your browser with
the address http://localhost:8000. But http://localhost (if it is not used)
should fail with a connection refused.

Step 2, forwarding localy port 80 to port 8000. This can be done with `ipfw`. This
is a command a little dark, but it works:

	sudo ipfw add 100 fwd localhost,8000 tcp from any to any 80 in

Basically, it just tells `ipfw` to forward traffic coming to any addresses of your
computer (ethernet port, virtual port, wifi connection etc...) to localhost port 8000.

We checked just the step before that the localhost:8000 was redirecting the
traffic to the virtual box port 80. We then now have a double redirection :

	<your computer> port 80 -> localhost port 8000 -> <virtual box> port 80

Your browser should give you a good news now like :

	It works!
	This is the default web page for this server.
	The web server software is running but no content has been added, yet.

Default message of Apache.


Wordpress
---------

For wordpress, the final step would be to setup `/etc/hosts` on the computers that
need to access your computer. This line will be enough :

	<IP address of your computer on the common network> <domain name of the wordpress> www.<domain name of the wordpress>

