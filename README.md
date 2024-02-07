# gol_dante_wfs
This is just a repo to store notes about creating a linux machine for our Dante enabled WFS system.

# Marian Driver for linux:

<https://github.com/MARIAN-Digital-Audio-Electronics/MARIAN-ALSA-DRIVER>

After following instructions, I saw an error that vmlinus cannot be found, but the card works despite this error/warning.

jackd start

```bash
jackd -R -p4096 -dalsa -r48000
``` 

# Extending jackd driver port limit:

In the initial attempt I came acros a limit built into Jack Audio Connect Kit.
There is a limit in jackd, called #define DRIVER_PORT_NUM, which by default is 256. This will not work with Marian Clara E, since it has many more ports.

To change this limit:
* Download jack2 source from github.
* Change the number 256 to 1024
* remove any existing jack2 packages that are already there (also jack1, which is included with debian). I just did the rather aggresive:

```bash
sudo apt remove jack*
```

Which is probably not very safe thing to do, I don't recommend this, but it worked for me.

Compile jack2 by going to the jack root folder and running:

```bash
./waf configure
./waf
sudo ./waf install
```

You may need to install a dependency to also have the ALSA driver for Jack included otherwise you will see an error when you start jackd.
You can check the build config options by doing ./waf --help

After this, you can install supercollider package as normal, it has jackd as a dependency (also qjackctl), but this doesn't override the custom jackd client in 
__/usr/local/bin__ so it seems to be ok.

The WFS library can be installed as a quark and works out of the box.

# WFS Settings

Remove master server and only use WFS servers.
You'll want to have as many servers as cores in the CPU.

# UDP memory limit change.

There seems to be a problem with buffer size of UDP messages in linux.
However, this buffer can be enlarged with the following command.

```bash
sudo sysctl -w net.core.rmem_max=13631488
sudo sysctl -w net.core.rmem_default=13631488
```

It also has the nice sideeffect of fixing the IDLE CPU rate to something more sensible. 


# Tracking CPU usage

top -n 1 -H -p 

this will show actual CPU usage by all jackd threads
top -H -p $(pgrep jackd)

taskset -apc 0-23 $(pgrep jackd)
