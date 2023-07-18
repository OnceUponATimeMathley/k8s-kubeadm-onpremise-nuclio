# Documentation for configuring traefik to expose some internal services

## Quick start ⚡
- [Traefik Dashboard](#traefik-dashboard)
- [Nuclio Dashboard](#nuclio-dashboard)

## TRAEFIK DASHBOARD
Here, we are using basic authentication with username and password when accessing the traefik dashboard.
1. Create secret for username and password
    - Install `apache2-utils` with `apt-get install apache2-utils`
    - Create username and password with `htpasswd`. <br>Run command `htpasswd -nb traefik traefik | openssl base64` (The username: traefik, password: traefik). <br>
    The output is `dHJhZWZpazokYXByMSR2UnNUZnQzTCRoalVOQi5XZkl2VHlBR2JONG5lRncuCgo=`
2. Create secret named `traefik-dashboard-auth` in `traefik` namespace with `kubectl apply -f traefik/secret/dashboard-basic-auth-secret.yaml`
3.  Create middleware named `traefik-dashboard-basicauth` in `traefik` namespace so when we access the traefik dashboard, it's forward to middleware and we must authenticate to access this dashboard. Command `kubectl apply -f traefik/middleware/basic-auth-middleware.yaml`
4. Create ingressroute named `traefik-dashboard` in `traefik` namespace with `kubectl apply -f traefik/ingressroute/traefik-dashboard-ingressroute.yaml`
5. Now, we can test this ingressroute by going to http://rainscales.com:32001/dashboard/ (We must add `rainscales.com 117.2.164.10` in `/etc/hosts` file)

## NUCLIO DASHBOARD
Similar to traefik dashboard, we create the ingressroute named `nuclio-dashboard` in `nuclio` namespace and the command is `kubectl apply -f traefik/ingressroute/nuclio-dashboard-ingressroute.yaml` <br><br>
After create ingressroute, we can test by going to http://rainscales.dashboard.com:32001/ (We must add `rainscales.dashboard.com 117.2.164.10` in `/etc/hosts` file)