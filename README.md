# Docker compose for unifi-controller

This is from [linuxserver/unifi-controller](https://docs.linuxserver.io/images/docker-unifi-controller)
whose [github repo is here](https://github.com/linuxserver/docker-unifi-controller).

The reason I needed this was because the unifi-controller I had installed using aur
on my Desktop machine stopped working. Probably because Arch Linux updates frequently
and the java default is now 11. Although I also have java-jre 8 installed the
unifi-controller still didn't working switchint to it.

So I looked around and using Docker seemed to be the "best" choice. There are
at least two choices. Doing a search for
[unifi-controller docker images](https://www.google.com/search?q=unifi-controller+docker+images)
the first choice is [linuxserver/unifi-controller](https://hub.docker.com/r/linuxserver/unifi-controller)
and the second choice is [jacobalberty/unify](https://hub.docker.com/r/jacobalberty/unifi).

I decide on using linuxserver because the documentation was clearer and it
was obvious for me how to use it, copy the example to a local `docker-compose.yml` file
and start it running. **But do not run with `docker-compose run --rm unifi-controller`** 

## Start the unifi-controllwer with `up`

The proper way to start `unifi-controller` is to use `docker-compose up`
and you'll see 
```
wink@3900x:~/docker/repos/linuxserver-unifi-controller
$ docker-compose up
Creating network "linuxserver-unifi-controller_default" with the default driver
Pulling unifi-controller (ghcr.io/linuxserver/unifi-controller:)...
latest: Pulling from linuxserver/unifi-controller
ea923265995f: Pull complete
f2e2a011e0f7: Pull complete
e05a5092b542: Pull complete
77461ae7ff50: Pull complete
9864911516ba: Pull complete
c1f225051185: Pull complete
f8d04f8e9995: Pull complete
933e447c7b41: Pull complete
21f88244f70f: Pull complete
Digest: sha256:3fe87a18b8384255e1616a1e73d80bea8e69671e9ae881f248cabc0be4cd50d2
Status: Downloaded newer image for ghcr.io/linuxserver/unifi-controller:latest
Creating unifi-controller ... done
Attaching to unifi-controller
unifi-controller    | [s6-init] making user provided files available at /var/run/s6/etc...exited 0.
unifi-controller    | [s6-init] ensuring user provided files have correct perms...exited 0.
unifi-controller    | [fix-attrs.d] applying ownership & permissions fixes...
unifi-controller    | [fix-attrs.d] done.
unifi-controller    | [cont-init.d] executing container initialization scripts...
unifi-controller    | [cont-init.d] 01-envfile: executing... 
unifi-controller    | [cont-init.d] 01-envfile: exited 0.
unifi-controller    | [cont-init.d] 10-adduser: executing... 
unifi-controller    | 
unifi-controller    | -------------------------------------
unifi-controller    |           _         ()
unifi-controller    |          | |  ___   _    __
unifi-controller    |          | | / __| | |  /  \ 
unifi-controller    |          | | \__ \ | | | () |
unifi-controller    |          |_| |___/ |_|  \__/
unifi-controller    | 
unifi-controller    | 
unifi-controller    | Brought to you by linuxserver.io
unifi-controller    | -------------------------------------
unifi-controller    | 
unifi-controller    | To support LSIO projects visit:
unifi-controller    | https://www.linuxserver.io/donate/
unifi-controller    | -------------------------------------
unifi-controller    | GID/UID
unifi-controller    | -------------------------------------
unifi-controller    | 
unifi-controller    | User uid:    1000
unifi-controller    | User gid:    1000
unifi-controller    | -------------------------------------
unifi-controller    | 
unifi-controller    | [cont-init.d] 10-adduser: exited 0.
unifi-controller    | [cont-init.d] 20-config: executing... 
unifi-controller    | [cont-init.d] 20-config: exited 0.
unifi-controller    | [cont-init.d] 30-keygen: executing... 
unifi-controller    | [cont-init.d] 30-keygen: exited 0.
unifi-controller    | [cont-init.d] 90-custom-folders: executing... 
unifi-controller    | [cont-init.d] 90-custom-folders: exited 0.
unifi-controller    | [cont-init.d] 99-custom-scripts: executing... 
unifi-controller    | [custom-init] no custom files found exiting...
unifi-controller    | [cont-init.d] 99-custom-scripts: exited 0.
unifi-controller    | [cont-init.d] done.
unifi-controller    | [services.d] starting services
unifi-controller    | [services.d] done.
```

Or run `detached` passing the `-d` flag:
```
wink@3900x:~/docker/repos/linuxserver-unifi-controller
$ docker-compose up -d
Creating network "linuxserver-unifi-controller_default" with the default driver
Creating unifi-controller ... done
```

## Stop the unifi-controller with `down`

```
wink@3900x:~/docker/repos/linuxserver-unifi-controller
$ docker-compose down
Stopping unifi-controller ... done
Removing unifi-controller ... done
Removing network linuxserver-unifi-controller_default
```

## Lessons learned

As mentioned above do not use `docker-compose run` instead
read the [linuxserver/docker-compose documentation]()
and use `docker-compose up | donw` as mentioned above.

To read about that long story follow my
[post to the linuxserver Discord channel](https://discordapp.com/channels/354974912613449730/506925392603512839/857446535011500032)
The short story is [Adam Beardwood](https://www.linkedin.com/in/adam-beardwood-251105a/),
aka [TheSpad](https://github.com/TheSpad) [directed me to use](https://discordapp.com/channels/354974912613449730/506925392603512839/857625468475801670)
`ss -4ntlp` to definitively determine that the ports weren't EXPOSED
to the host after I started unifi-controller when using
`docker-compose run --rm unifi-controller`. In particular, below we see
none of the ports enumerated in the docker-compose.yml file. Most
importantly port 8443 is not listed:
```
wink@3900x:~
$ ss -4ntlp
State    Recv-Q   Send-Q     Local Address:Port      Peer Address:Port   Process                               
LISTEN   0        511            127.0.0.1:6463           0.0.0.0:*       users:(("Discord",pid=2596,fd=96))   
LISTEN   0        128              0.0.0.0:7396           0.0.0.0:*                                            
LISTEN   0        4096           127.0.0.1:27017          0.0.0.0:*                                            
LISTEN   0        128              0.0.0.0:36330          0.0.0.0:*                                            
LISTEN   0        128              0.0.0.0:22             0.0.0.0:*  
```
Also when you do a `docker ps` the ports are "single-sided" (my term).
In the `docker ps` below when it's working we see ports of the form
`0.0.0.0:8443->8443/tcp` showing the actual maping of the two sides:
```
wink@3900x:~/docker/repos/linuxserver-unifi-controller
$ docker ps
CONTAINER ID   IMAGE                                  COMMAND   CREATED         STATUS         PORTS                                    NAMES
dbe5d4cff188   ghcr.io/linuxserver/unifi-controller   "/init"   6 minutes ago   Up 6 minutes   8080/tcp, 8443/tcp, 8843/tcp, 8880/tcp   linuxserver-unifi-controller_unifi-controller_run_65b8d68ac8c8
```

What Adam directed me to do was to `down` container:
```
wink@3900x:~/docker/repos/linuxserver-unifi-controller
$ docker-compose down
Stopping linuxserver-unifi-controller_unifi-controller_run_3ecb3dfdde6a ... done
Removing linuxserver-unifi-controller_unifi-controller_run_3ecb3dfdde6a ... error

ERROR: for linuxserver-unifi-controller_unifi-controller_run_3ecb3dfdde6a  removal of container 14d27b6bd2f5b21144b316349ba23307eb74f40b5f28acaa02b05e021eafd18f is already in progress
Removing network linuxserver-unifi-controller_default
```
And then bring it backup `up`, which rebuilds it:
```
wink@3900x:~/docker/repos/linuxserver-unifi-controller
$ docker-compose up -d
Creating network "linuxserver-unifi-controller_default" with the default driver
Creating unifi-controller ... done
```
Now running `ss -4ntlp` we do see all of the ports as seen in the docker-compose.yml
file. Especially `8443`, the primary unifi-controller web interface port:
```
wink@3900x:~
$ ss -4ntlp
State    Recv-Q   Send-Q     Local Address:Port      Peer Address:Port   Process                               
LISTEN   0        4096             0.0.0.0:8443           0.0.0.0:*                                            
LISTEN   0        511            127.0.0.1:6463           0.0.0.0:*       users:(("Discord",pid=2596,fd=96))   
LISTEN   0        128              0.0.0.0:7396           0.0.0.0:*                                            
LISTEN   0        4096             0.0.0.0:6789           0.0.0.0:*                                            
LISTEN   0        4096           127.0.0.1:27017          0.0.0.0:*                                            
LISTEN   0        128              0.0.0.0:36330          0.0.0.0:*                                            
LISTEN   0        4096             0.0.0.0:8843           0.0.0.0:*                                            
LISTEN   0        4096             0.0.0.0:8080           0.0.0.0:*                                            
LISTEN   0        4096             0.0.0.0:8880           0.0.0.0:*                                            
LISTEN   0        128              0.0.0.0:22             0.0.0.0:*
```
And now when I do a `docker ps` we see "double-sided" ports:
```
$ docker ps
CONTAINER ID   IMAGE                                  COMMAND   CREATED             STATUS             PORTS                                                                                                                                                                                                                                                                                                                                                                                                   NAMES
94f1a7a4c577   ghcr.io/linuxserver/unifi-controller   "/init"   About an hour ago   Up About an hour   0.0.0.0:1900->1900/udp, :::1900->1900/udp, 0.0.0.0:3478->3478/udp, :::3478->3478/udp, 0.0.0.0:6789->6789/tcp, :::6789->6789/tcp, 0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:8443->8443/tcp, :::8443->8443/tcp, 0.0.0.0:8843->8843/tcp, :::8843->8843/tcp, 0.0.0.0:5514->5514/udp, :::5514->5514/udp, 0.0.0.0:10001->10001/udp, :::10001->10001/udp, 0.0.0.0:8880->8880/tcp, :::8880->8880/tcp   unifi-controller
```

Another thing I learned was that you can connect to a container
with multiple terminals using `docker exec it <id | name of container> bash`.
I used this to verify that I could use curl to verify the unifi-controller
```
wink@3900x:~/docker/repos/linuxserver-unifi-controller
$ docker exec -it unifi-controller bash
root@213e55943623:~
# curl -v https://127.0.0.1:8443
* Rebuilt URL to: https://127.0.0.1:8443/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (OUT), TLS alert, Server hello (2):
* SSL certificate problem: self signed certificate
* stopped the pause stream!
* Closing connection 0
curl: (60) SSL certificate problem: self signed certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

