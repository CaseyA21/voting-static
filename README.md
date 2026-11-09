# Polls

In this implementation of the Voting application, the `.html`, `.css`, and `.js` files are served up separately from the Flask API.  The Flask server and the DynamoDB Local database still run on the same instance.  

When run locally, one web server will provide the static content (`.html`, `.css`, and `.js` file) on poart 8080, and a second web server will run on port 5000 to serve the API.  

On AWS the static content will be provided via an S3 website and the API/database will run on an EC2 instance.

In this version, when the javascript of the application makes a call to the API, it is considered a *cross-origin domain* request.  Web browsers will block this type of request unless the second domain explicitly allows it.  In this case, we have to configure the Flask server to allow cross-origin requests.  This system uses the  [`flask-cors`](https://flask-cors.readthedocs.io/en/latest/) library to configure the required CORS headers.

The API sends `Access-Control-Allow-Origin: *`. That header tells the browser that any website may read the API's responses. It fits this service, which has no login and sets no cookies. It is a poor policy for a system that keeps a session: that system names the one site allowed to call it, and it does not use `*`.

`ui/config.js` sets `DEPLOY_URL`. The pages call that address, except when they are served from `127.0.0.1` or `localhost`, where they call `http://127.0.0.1:5000`. `.env` is the API's configuration for DynamoDB. The pages do not read it.

See the **specs** repository for the product specifications (use cases, API, data model, and related materials).


## Documentation

| Doc | Contents |
|-----|----------|
| [Development setup](docs/development.md) | Virtualenv, install, unit tests, lint, `.env`, DynamoDB Local, run the API and the pages, acceptance tests |
| [Deploy on EC2](docs/deploy-ec2.md) | User data, systemd, debugging |
| [Deploy with CloudFormation](docs/deploy-cloudformation.md) | CLI: create stack from `deploy/cloudformation.yaml` |
| [Deploy the pages](docs/deploy-s3.md) | Set `DEPLOY_URL` and upload `ui/` to the website bucket |
| [Design](docs/design.md) | Pages and the API, layers, data types |



## Quick start

- Create a virtual environment and install dependencies
- Configure `.env` from `config/example.env`
- Start DynamoDB Local and create the `Polls` table
- Run `python -m voting.app` and `python -m http.server 8080 --directory ui`
- Open [http://127.0.0.1:8080/](http://127.0.0.1:8080/)



See [Development setup](docs/development.md) for full steps.
