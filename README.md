#container, a plugin for dokku

Container is a very simple plugin for [dokku](https://github.com/progrium/dokku) the docker powered mini-Heroku and smallest PaaS implementation you've ever seen.

##the problem

Dokku keeps a file named CONTAINER with the docker container id for every application. Most [dokku plugins](https://github.com/progrium/dokku/wiki/Plugins) require this file including the built in `delete` command to remove a previously installed application. On occasion, especially when a build failed, this file may not have been created yet.

It is not always possible to avoid a build failure as you may find yourself in a "chicken or egg" scenario where your app depends on external services like a database and will not deploy without these. To create these services you may recuire an existing application. Because your build failed dokku did create the app but not, the CONTAINER file a crucial piece of the puzzle.

You may see errors like:

    /home/dokku/example-app/CONTAINER: No such file or directory

Simply touching the missing file will cause several docker help messages in the output like:

    Usage: docker kill [OPTIONS] CONTAINER [CONTAINER...]

    Kill a running container (send SIGKILL, or specified signal)

      -s, --signal="KILL": Signal to send to the container

Which is the result of calling `docker kill` without a container id, in this example.  
  
##solution

More of a work around really, doctoring the symptoms instead of fixing the cause, but in the meantime until we find a permanent cure, this allows you to get on with deploying your application.

The problem is solved by creating the CONTAINER file based on the last created container id. We can retrieve the information for this container with the `docker ps` command.

    $ docker ps -n 1
    CONTAINER ID        IMAGE                    COMMAND             CREATED            STATUS        PORTS         NAMES
    99f04e250632        app/example-app:latest   /build/builder      2 minutes ago      Exit 1                      kickass_newton

If you have access you can simply add the container id `99f04e250632` to `$DOCKER_HOME/$APP/CONTAINER` but this rather defeats the purpose.

##installation

Standard dokku plugin installation will suffice:

    $ cd /var/lib/dokku/plugins
    $ git clone <git url>
    $ dokku plugins-install

##usage

The **container** plugin exposes one simple command **container:create** by which you can remotely created the container file for an application **if it does not already exist** based on the container id of the last created container as reported by the command `docker ps -n 1` by doing something similar to:

    $ ssh dokku container:create example-app

Assuming you have the following section configured in your ssh client config file (located at `~/.ssh/config`)

```ssh
Host dokku
   HostName dokku.example.com
   User dokku
   RequestTTY yes
```

You can also run the `dokku` command directly on the server but unless it's run as the `dokku` user the `CONTAINER` file will likely be created with incorrect privileges and is therefor not recommended.

##improve

It is very likely that this project will be short lived as we would probably see a more permanent fix upstream short for long. Should you have any ploblems, suggestions or improvements your contributions are welcomed. 

##license

Use, learn, abuse, and share alike, `copyright © 2014 nickl- en kie` as per included `BSD (3-clause)` license.

##disclaimer

```
THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
