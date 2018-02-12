# docker-belos-cicd

Docker microservice for BelOS Continuous Integration and Continous Deployment

## Why

Because I always believe that even advanced technologies should be available to everyone. This is a tentative to offer everyone the possibility to build his own images of BelOS, when he wants, how he wants. You don't have to wait anymore for build bots. No more scene drama. Just build and enjoy your favourite Android ROM.

## Why Docker?

Because I'm a big fan of isolating everything if possible. I don't want to reinstall my OS or triage with dirty packages, just because today I need somethng, and tomorrow I'll need something else.

## Requirements

- At least Dual Core CPU ( Higher is better )
- At least 6GB RAM ( Higher is better )
- At least 200GB HDD Space ( Higher is better )

### Android propretary binaries

By default when you build Android from scratch you need to pull the Binaries of your interested device via ADB. Although via this Docker is not possible to do so ( would imply having all the devices connected to that machine and ideally know how to switch from one to the other before pulling ).

In order to enable the CICD pipeline to work, I highly suggest to make use of this manifest (https://github.com/TheMuppets/manifests) and save it inside your mapped `/srv/local_manifests` folder, by doing for example:
```shell
$ curl -o https://raw.githubusercontent.com/TheMuppets/manifests/cm-14.1/muppets.xml /srv/local_manifests/muppets.xml
```

## How it works

This docker will autobuild any device list given for a specified branch every midnight at 02:00 UTC. In the end, any built ZIP will be moved to the relative volume mapped directory to `/srv/zips`.

> **IMPORTANT:** Remember to use VOLUME mapping. By default Docker creates container with max 10GB of Space. If you will not map volumes, the docker will just break during Source syncronization!

## Configuration

You can configure the Docker by passing custom environment variables to it. See the [Dockerfile](Dockerfile#L11) for more details.

## How to use

### Simple mode
build BelOS 2 for `hammerhead` with default settings
```
docker run \
    --restart=always \
    -d \
    -e "USER_NAME=John Doe" \
    -e "USER_MAIL=john.doe@awesome.email" \
    -e "DEVICE_LIST=hammerhead" \
    -v "/home/user/ccache:/srv/ccache" \
    -v "/home/user/source:/srv/src" \
    -v "/home/user/zips:/srv/zips" \
    danialbehzadi/docker-belos-cicd
```

### Advanced mode
build cm-13.0 LineageOS for `hammerhead` and `bullhead`
```
docker run \
    --restart=always \
    -d \
    -e "USER_NAME=John Doe" \
    -e "USER_MAIL=john.doe@awesome.email" \
    -e "OS_URL=git://github.com/lineageos/android.git"
    -e "BRANCH_NAME=cm-13.0" \
    -e "DEVICE_LIST=hammerhead,bullhead" \
    -v "/home/user/ccache:/srv/ccache" \
    -v "/home/user/source:/srv/src" \
    -v "/home/user/zips:/srv/zips" \
    danialbehzadi/docker-belos-cicd
```

### Expert mode
build staging Belos 2 for a device that doesn't exist inside the main project, but comes from a special manifest ( has to be created inside `/home/user/local_manifests/` ). Finally provide a custom OTA URL for this ROM so users can update using built-in OTA Updater.
```
docker run \
    --restart=always \
    -d \
    -e "USER_NAME=John Doe" \
    -e "USER_MAIL=john.doe@awesome.email" \
    -e "BRANCH_NAME=staging-2" \
    -e "DEVICE_LIST=n80xx" \
    -e "OTA_URL=http://cool.domain/api" \
    -v "/home/user/ccache:/srv/ccache" \
    -v "/home/user/source:/srv/src" \
    -v "/home/user/zips:/srv/zips" \
    -v "/home/user/local_manifests:/srv/local_manifests" \
    danialbehzadi/docker-belos-cicd
```
**NOTE:** `/home/user/local_manifests/` may contain multiple XMLs, since all the files will be then copied inside `.repo/local_manifests/`
