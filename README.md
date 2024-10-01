This is a repo composed of two submodules, a NodeJS backend and a SvelteKit frontend.
The NodeJS backend currently connects to an external MongoDB database.
This repo contains the information to run and deploy both. This includes a Traefik server to handle SSL termination and obtaining and renewing SSL certificates from letsencrypt.

Certificates are stored automatically in the folder /letsencrypt

Before running, you should set up a hashed password file for the Traefik Dashboard. To do so, run:
echo "admin:$(openssl passwd -apr1 strongpassword)" > traefik-dashboard-auth/.htpasswd

Replacing strongpassword with a strong password. This will store login information used by Traefik in 
traefik-dashboard-auth/.htpasswd

To deploy, run:
docker-compose up --build -d

The Traefik dashboard will be reachable via 
https://scribe.dominikkoller.com:8080
(given that is the domain you deploy to - otherwise you'll need to change the docker-compose.yml file though)
Make sure your 8080 port is opened, eg in your EC2 instance configuration.

To deploy locally, run
docker-compose -f docker-compose.local.yml up --build

For your local HTTPS connection, you need to creat local certificates:
mkdir certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/privkey.pem -out certs/fullchain.pem \
  -subj "/CN=scribe.localhost"

Also, add the following to your /etc/hosts/ file:
127.0.0.1 scribe.localhost

Then, the services will then be available under
https://scribe.localhost/
make sure to type https, as before you have accepted your local certificate your browser might not allow a auto redirect to https.

The browser of the SvelteKit frontend communicates directly with the backend.
This is to decouple the SvelteKit application from the backend - it could be hosted anywhere independently, or replced with any other client that has no hosting server.

Both projects contain their own .env file
Adapt them to your needs.

The backend .env must contain:
EXPRESS_PORT=3001
HOCUSPOCUS_PORT=3002

MONGODB_URI=mongodbURI
JWT_SECRET=jwt_secret (!!! REPLACE THIS WITH A JWT SECRET)
OPENAI_API_KEY=openaikey

The frontend .env must contain (these for local development, use the actual domain in production):
PUBLIC_API_BASE_URL=https://scribe.localhost/api/
PUBLIC_APOLLO_URL=https://scribe.localhost/graphql
PUBLIC_HOCUSPOCUS_URL=https://scribe.localhost/hocuspocus

These docker containers are currently hosted on a AWS EC2.