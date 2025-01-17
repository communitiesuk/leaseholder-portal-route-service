# GOV.UK PaaS IP authentication route service

This repo contains a simple Nginx application which acts as a proxy for your
web applications and provides an IP restriction layer.

This repo is a template, which you should customise according to your needs
using the application manifest.

All PaaS traffic will go through the route service to filter traffic.

## Requirements

- Cloud Foundry CLI (https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)

You should log in using the Cloud Foundry CLI
(https://docs.cloud.service.gov.uk/#setting-up-the-command-line).

For all actions you should always make sure you are targeting the correct
space.

## Customisation

Edit the `manifest.yml` and change the `ALLOWED_IPS` as appropriate.

## Deployment by hand

To deploy the app, run `cf push`.

If you have not overwritten the `((app-name))` variables then you will need to
run `cf push --var app-name=my-app`

If you want to add a custom route, add a route definition to the manifest:

```applications:
  - name: ((app-name))
    routes:
      - route: my-subdomain.my-domain.com
    ...
```

## Deployment by script

This repository provides a script that will deploy and configure this route service for you. See the example below for how to use it.

```shell script
ALLOWED_IPS="comma_separated_list_of_ips_eg_1.2.3.4,5.6.7.8" \
ROUTE_SERVICE_APP_NAME="name_of_the_app_to_push" \
ROUTE_SERVICE_NAME="name_of_the_route_service_to_create" \
PROTECTED_APP_NAME="name_of_the_app_to_protect" \
./deploy.sh
```

## Use the app as a route service

Please refer to the official GOV.UK PaaS
[documentation on route services](https://docs.cloud.service.gov.uk/deploying_services/route_services/#user-provided-route-services)
for steps on deploying the route service.

## Checking that it works

The route service exposes two paths for checking the status.

The path `/_route-service-health` is for information and health checking, and
has stats about the number of active connections which exist.

The path `/_route-service-check` is for checking if you may use the route
service. If you are, then you will receive `OK`, if you are not you will
received `Forbidden by ((app-name))`, where `((app-name))` is the value of the
`APP_NAME` environment variable.

## Updating the IP Whitelist

### Getting the Current Whitelisted IPs

The currently whitelisted IPs can be retrieved from an environment variable in the **leaseholder-portal-route-service-app** in GDS PaaS. To retrieve them:

1.  Login to CloudFoundry:

        $ cf login

2.  Target the **leaseholder-portal-production** space:

        $ cf target -s leaseholder-portal-production

3.  Get the currently whitelisted IPs from **leaseholder-portal-route-service-app**:

        $ cf env leaseholder-portal-route-service-app | grep ALLOWED_IPS

### Whitelisting a New IP Address

To whitelist a new IP address make sure you are targetting the **leaseholder-portal-production** space then run the following command with all the existing whitelisted IP addresses, and add the new one at the end:

```shell script
ALLOWED_IPS="IP_1,IP_2,...,IP_N" \
ROUTE_SERVICE_APP_NAME="leaseholder-portal-route-service-app" \
ROUTE_SERVICE_NAME="leaseholder-portal-route-service" \
PROTECTED_APP_NAME="leaseholder-portal" \
./deploy.sh
```

For example:

```shell script
ALLOWED_IPS="1.2.3.4/32,5.6.7.8/24" \
ROUTE_SERVICE_APP_NAME="leaseholder-portal-route-service-app" \
ROUTE_SERVICE_NAME="leaseholder-portal-route-service" \
PROTECTED_APP_NAME="leaseholder-portal" \
./deploy.sh
```

### Disable IP Whitelist

```shell script
ALLOWED_IPS="all" \
ROUTE_SERVICE_APP_NAME="leaseholder-portal-route-service-app" \
ROUTE_SERVICE_NAME="leaseholder-portal-route-service" \
PROTECTED_APP_NAME="leaseholder-portal" \
./deploy.sh
```
