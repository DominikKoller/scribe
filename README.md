This is a repo composed of two submodules, a NodeJS backend and a SvelteKit frontend.
The NodeJS backend currently connects to an external MongoDB database.

To start both, run 
docker-compose up --build

The browser of the SvelteKit frontend communicates directly with the backend
This could be changed so we do not have to expose the backend's address at all

Both projects contain their own .env file
Adapt them to your needs.

The backend .env must contain:
EXPRESS_PORT=3001
HOCUSPOCUS_PORT=3002

MONGODB_URI=mongodbURI
JWT_SECRET=jwt_secret
OPENAI_API_KEY=openaikey

The frontend .env must contain (these for local development, or full IPs):
PUBLIC_API_BASE_URL=http://localhost:3001/api
PUBLIC_HOCUSPOCUS_URL=http://localhost:3002