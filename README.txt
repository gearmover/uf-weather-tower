University of Florida Weather Tower Control Software 					v 0 . 1
--------------------------------------------------------------------------------------------------------------------------------------------------------
Lead Developer : Chris Pergrossi
Asst Developer : Travis Cole

---------
Abstract|
---------

This software is the brains behind the Powell Center for Hurricane Research Inclement Weather Sensor Towers, controlling collection of data from the multitude
of sensors that cover these five towers.  The software is written using National Instrument's LabVIEW Data-Driven Programming Language and is being developed for
embedded systems - the N.I. cRIO 9012 Controller along with attached FPGA and several additional modules.  This software is being developed for educational use by
the Powell Research Lab, and as so much as I can, I release the source for anyone to use as they see fit, just mention the University of Florida and Powell Labs as
the original developers in an appropriate credits or acknowledgements.

---------
LEGALESE|
---------

That being said, I do NOT declare this software fit for any purpose including the one for which it is being developed!  It is and will continue to be a work in progress,
I've made no claims about its fitness for your project or the availability of the developer for technical support - USE THIS CODE AT YOUR OWN RISK!  I, the University of Florida, 
Powell Labs, my coffee bean place, EVERYONE I haven't mentioned but had a finger in this project is not to be held liable for anything, damages or otherwise, from
the use, misuse, or abuse of this software.  I GUARANTEE there are bugs in the code, but I'm fixing them as quickly as I can while also adding functionality - not an easy task!

That being said, I'm quite proud of the current state of the project - it's coming along rather nicely!

----------
SPECIFICS|
----------

The code is broken into 3 main sections:
	1. The Hypervisor : as its name implies, this is the head honcho of the software, his is the VI that you'll 'RUN' - he'll dynamically read a config file (ReadConfig.vi) and 
						parse the modules passed in.  After parsing, the hypervisor  proceeds to load the modules, and then run in a loop servicing all the threads.  By servicing, I mean 
						it will iterate through each thread, check if it's still alive, and send a "Heartbeat" message to let the module the hypervisor is still running (mostly a DEBUG item)

	2. The Producer Modules : These are extremely cheap threads that share a common data queue. These threads would poll the actual sensors on the Towers and push their data into a queue
								structure.  Their asynchronous nature lends itself to easy polling management (i.e. poll at 10 Hz, just include a WaitToNextMultipleMS(100)) because they
								are all completely independent of each other.  However, these producer modules can perform a lot of other functions besides just sensor polling.  For example,
								we could create a WatchDog module, that polls the hypervisor state (or any other value) and records that as data (another example of a WatchDog - "Is it safe
								to return to the tower?").  Another interesting idea would be to run a web server as a producer module, and send the received requests as do-not-log data to a 
								special consumer that handles the actual data requests (consumer modules are next).  Yet another example of a "producer" module would be a batch server upload thread;
								a module that monitors the [C] Log To Disk consumer module, dictating where on disk to write and monitoring the file size, keeping the Database files under a certain size.
								When a database file is completed, the [P] Batch Upload module can ZIP compress the file and upload to FTP or WebDAV.

	3. The Consumer Modules : These are very EXPENSIVE threads that cause a seperate copy of ALL the producer data each iteration to each consumer.  The reason being compartmentalization; we don't 
							  want one consumer misbehaving and eating all the data before it can be logged away.  Currently implemented consumer modules are an early draft (not master-slave as described
							  in the producer section) of the [C] Log To Disk module, and another early edition of the [C] Upload to Server byte-by-byte upload (very inefficient) - and required a server 
							  module I wrote in Python to be running (it is very quick and dirty Python too!) in order to accept the data and log it to file on the remote server.