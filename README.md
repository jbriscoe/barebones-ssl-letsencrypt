# Getting a malware free, SSL certified website up as quick as possible
Because reasons? 🤷

With thanks to this guide: https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion/wiki/Basic-usage 

The goal of this is to hand off the creation, and renewal of the SSL certificate to a seperate container that does the lifting for you. It then proxies in one or multiple containers that you give it.

## Step 1:
`docker run --detach \
    --name nginx-proxy \
    --publish 80:80 \
    --publish 443:443 \
    --volume /etc/nginx/certs \
    --volume /etc/nginx/vhost.d \
    --volume /usr/share/nginx/html \
    --volume /var/run/docker.sock:/tmp/docker.sock:ro \
    jwilder/nginx-proxy`

## Step 2:

`docker run --detach \
    --name nginx-proxy-letsencrypt \
    --volumes-from nginx-proxy \
    --volume /var/run/docker.sock:/var/run/docker.sock:ro \
    jrcs/letsencrypt-nginx-proxy-companion`

--volumes-from is amazing! it inherits all the volumes from another place, so you now have a shared place to pass your certs and configs between.

## Step 3: 
Start providing one or more containers to be used. Each one needs to internally use a unique port, that is not 80 or 443.

In this folder, There is a sample one ready to try out that will spin up it's own nginx and index html.
### `docker build . -t yourbuiltdockertaghere`
The tag you give it is important, because we will use it again later.

`docker run --detach --name burgertime -p 37000:37000 --env "VIRTUAL_HOST=isitburgertimeyet.club" --env "VIRTUAL_PORT=37000" --env "LETSENCRYPT_HOST=isitburgertimeyet.club" --env "LETSENCRYPT_EMAIL=your@email.com" yourbuiltdockertaghere`

You need to expose the port on your container for it to be picked up by the listener for new ports in the docker container.

Then go eat burgers.
