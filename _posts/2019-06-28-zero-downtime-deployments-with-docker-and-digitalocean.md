---
layout: post
title: "Zero Downtime Deployments with Docker and DigitalOcean"
photo: docker
meta: post
---

![](/images/docker.jpg)

Previously, [Explorable Places][ep] used Capistrano to deploy to one droplet in DigitalOcean. This setup had a number of downsides and limitations. Among them,<!--more--> we had:
- a single point of failure
- only _nearly_ zero downtime
- no easy way to scale horizontally
- difficulty in upgrades

When the app started out, it was easy and only necessary to have one box up that hosted everything: nginx, app, db and cache (memcached). Provisioning a new droplet would be a pain in the butt, and I had no way to distribute traffic to each box anyway. Platform upgrades (from Ubuntu 12 to 14) proved to be a royal pain in the behind, and required the site to be down for some time, in addition to the risk involved in the update itself.

## Docker

Dockerizing the app would solve a couple of our issues, and enable us to solve the others. There are multiple articles out there about how to convert your app, no matter the stack, to Docker, so I won't go into such a large topic here. But for me, the basic steps were:

- Move DB out of our droplet, and into a managed host
- Dockerize our app with docker-compose, and utilize Dockerhub to manage our images
- Create a couple Docker droplets from the DigitalOcean marketplace, managed under a DigitalOcean load balancer
- Write a deploy script that will deploy to each environment

## Deploying

Now that you presumably have your Dockerized app, along with all your separate `docker-compose` files, environment variable files, and others that you do not want checked into source countrol, we need a way to get those files to our droplets, and start up our app.

Below, I've outlined our deploy script. See the bottom of this post for all the gory details, but what it does is:
1. Use [DropletKit][droplet_kit] to get a list of our staging/production droplets
1. Loop over each droplet to do the following:

1. Take the droplet out of traffic with the load balancer
1. Copy all environment files, docker-compose files, etc. to the droplet.
1. Run a start up script with the droplet that:
    * Take down all running containers and remove them, as well as volumes, and images.
    * Pull in latest tagged image from DockerHub
    * Bring up containers
    * Put back into traffic with the load balancer if all was successful

With this in place, you can simply deploy your app with:

{% highlight shell %}
./deploy.rb staging
# or
./deploy.rb production
{% endhighlight %}

## Health Checks

The one key here is that we must be certain that our server is up and serving traffic before we add it back to the load balancer to receive traffic. By default, Docker Compose will consider the container to be started and healty as soon as it is up, even if it takes 10-15 seconds for your server within your container to start.

To make sure our app is ready to serve traffic, we can utilize health checks with Docker Compose. You can define this in your compose file like so:

{% highlight yaml %}
version: "2.1"
services:
  app:
    healthcheck:
      test: curl --fail -s http://localhost:3000/healthcheck || exit 1
      interval: 5s
      timeout: 60s
      retries: 10
  depends_on:
      app:
        condition: service_healthy
{% endhighlight %}

This health check will be run within your container, so you can reference localhost here. If you have other containers that depend on this one, you can define the conditions that will dictate when that container is brought up. Above, we have an nginx service that depends on our app. We tell it to wait until our app is healthy, which will depend on the health check.

Health checks will continue to run, so be sure to exlude it from any:
- SSL validation
- Performance monitoring (will severely skew your 'typical' and 'problem' response times)
- IP throttling (don't want to throttle yourself)

## Scripts (gory details)

### Deploy script

{% highlight ruby %}
#!/usr/bin/env ruby

require 'rainbow'
require 'droplet_kit'
client = DropletKit::Client.new(access_token: ENV['DO_TOKEN'])

load_balancer_id = client.load_balancers.all.first.id

deploy_env = ARGV[0]
if deploy_env == 'production'
    droplet_ids = client.load_balancers.find(id: load_id).droplet_ids
    droplets = droplet_ids.map {|id| client.droplets.find(id: id) }
elsif deploy_env == 'staging'
    droplets = [client.droplets.find(id: ENV['STAGING_DROPLET_ID'])]
end

if !droplets
    raise("There are no droplets for deploy env: #{deploy_env}!")
end

droplet_hash = {}

# set statuses
droplets.each do |droplet|
    droplet_hash[droplet.name] = 'not deployed'
end

droplets.each do |droplet|
    if deploy_env == 'production'
        puts Rainbow("REMOVING #{droplet.name} FROM THE LOAD BALANCER").green
        client.load_balancers.remove_droplets([droplet.id], id: load_id)
    end

    puts Rainbow("COPYING OVER NGINX CONFIG TO #{droplet.name}").green

    # copy nginx
    # copy docker compose files and .env files
    system("scp ./docker/nginx/nginx.conf ./docker/nginx/nginx.#{deploy_env}.conf ./docker/nginx/ssl/#{deploy_env}.explorableplaces.crt ./docker/nginx/ssl/#{deploy_env}.explorableplaces.key docker-compose.yml docker-compose.#{deploy_env}.yml .env.common .env.#{deploy_env} root@#{droplet.public_ip}:~/")

    puts Rainbow("DEPLOYING NEW CONTAINERS TO #{droplet.name}").green

    puts "IP: #{droplet.public_ip}"
    puts "deploy: #{deploy_env}"
    deploySuccess = system({
        "DEPLOY_ENV" => deploy_env
    }, "(export -p; cat ./start-up.sh) | ssh root@#{droplet.public_ip}  'bash -s'")

    if deploySuccess
        puts Rainbow("SUCCESSFULLY DEPLOYED NEW CONTAINERS TO #{droplet.name}").green
        droplet_hash[droplet.name] = 'deployed'

        if deploy_env == 'production'
            client.load_balancers.add_droplets([droplet.id], id: load_id)
        end
    else
        droplet_hash[droplet.name] = 'deploy failed'
        puts Rainbow(droplet_hash).red
        raise("Did not execute deploy.sh script correctly")
    end
end

puts "FINISHED DEPLOY, STATUS:"
droplet_hash.each do |name, status|
  puts "#{name}: #{status}"
end
{% endhighlight %}

### Start up script
{% highlight shell %}
echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
docker-compose down --remove-orphans
docker system prune -f --all --volumes

echo "$DEPLOY_ENV"
if [ "$DEPLOY_ENV" == 'production' ]
then
    docker-compose -f docker-compose.yml -f docker-compose.production.yml pull && docker-compose -f docker-compose.yml -f docker-compose.production.yml up -d --no-build
elif [ "$DEPLOY_ENV" == 'staging' ]
then
    docker-compose -f docker-compose.yml -f docker-compose.staging.yml pull && docker-compose -f docker-compose.yml -f docker-compose.staging.yml up -d --no-build
fi
{% endhighlight %}

[ep]: https://www.explorableplaces.com
[droplet_kit]: https://github.com/digitalocean/droplet_kit
