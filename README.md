# Setup

Clone phabricator, libphutil, arcanist into this directory
 - `git clone https://github.com/phacility/phabricator.git`
 - `git clone https://github.com/phacility/arcanist.git`
 - `git clone https://github.com/phacility/libphutil.git`

Run phabricator with `docker-compose up`

`./firefox-proxy` launches an instance of firefox which will
proxy web requests into the docker network (Phabricator expects
to be accessed through a particular hostname). The tinyproxy
will be exposed as port 1080.

Access phabricator at http://phabricator.test
