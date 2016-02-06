Nginx-rtmp server with ffmpeg tools
===================================

This container provides a [nginx-rtmp](https://github.com/arut/nginx-rtmp-module) server listening on ports **8080** (http) and **1935** (RTMP). The container includes the ffmpeg binaries.

Usage
-----

From Docker registry:

```
docker pull rjrivero/nginx-rtmp
```

Or build yourself:

```
git clone https://github.com/rjrivero/docker-nginx-rtmp.git
docker build --rm -t rjrivero/nginx-rtmp docker-nginx-rtmp
```

Running the image:

```
docker run --rm -p 8080:8080 -p 1935:1935 --name nginx-rtmp rjrivero/nginx-rtmp
```

RTMP server configuration
-------------------------

To enable the streaming service, you will need to add at least one **application** stanza to the nginx configuration. You can do that mounting a configuration file under the */etc/nginx/conf.d/* path, for example:

```
docker run -d -p 8080:8080 -p 1935:1935 -v </path/to/my/rtmp.conf>:/opt/nginx/conf.d/rtmp.conf --name nginx-rtmp rjrivero/nginx-rtmp
```

Your **rtmp.conf** file must have an application configuration section like the following:

```
application myapp {
  live on;
}
```

This configuration allows you to push RTMP live streams to your server, using the *myapp* application name in the rtmp url: **rtmp://localhost:8080/myapp/mystream**. You can push a video using ffmpeg, for instance

```
ffmpeg -re -i <my_video.mpg> -c:v libx264 -c:a libmp3lame -ar 44100 -ac 1 -f flv rtmp://localhost/myapp/mystream
```

Then you can play the video with vlc:

```
vlc rtmp://localhost/myapp/mystream
```

You can also see the streaming stats pointing your browser to http://localhost:8080/stat

This is just scratching the surface of what nginx-rtmp can do. For the complete reference, visit the [project wiki](https://github.com/arut/nginx-rtmp-module/wiki), or have a look at the helpful tutorials at [The Helping Squad](http://www.helping-squad.com/?s=nginx), nice stuff there.

FFmpeg
------

ffmpeg binaries are located at */usr/local/bin/*.

Customization
-------------

You can customize the service dropping your files at the following paths:

  - **/opt/nginx/html/**: this is the HTML server root. You can mount your landing page here.

  - **/etc/nginx/conf.d/**: You can mount any configuration file here, and it will be included in the rtmp server section of nginx config. In fact, the rtmp server configuration in this container is just:

```
rtmp {
    server {
        listen 1935;
        ping 30s;
        notify_method get;

        # Everything else,m get from the config volume!
        include /opt/nginx/conf.d/*.conf;
    }
}
```

Volumes
-------

The RTMP server may need a large storage space for temporary files, and to store recordings. This space should be provided as a volume, mounted at **/opt/rtmp**. The server is configured to store several kinds of temp files under this path:

    - HTTP client-body temp files under /opt/rtmp/client_body,
    - Proxy temp files under /opt/rtmp/proxy,
    - FastCGI temp files under /opt/rtmp/fastcgi,
    - Uwsgi temp files under /opt/rtmp/uwsgi,
    - Scgi temp files under /opt/rtmp/scgi

You must make the *rtmp* volume writable to the **nginx** user, which is UID 1000, gid 1000. Before mounting the volume, change ownership of the folder to this user:

```
sudo chown -R 1000:1000 </path/to/your/storage/folder>

docker run --rm -p 8080:8080 -p 1935:1935 \
    -v </path/to/your/rtmp.conf>:/opt/nginx/conf.d/rtmp.conf:ro \
    -v </path/to/your/html/root>:/opt/nginx/html:ro \
    -v </path/to/your/storage/folder>:/opt/rtmp \
    rjrivero/nginx-rtmp
```

Ports
-----

The container exposes ports **8080** (HTTP) and **1935** (RTMP)
