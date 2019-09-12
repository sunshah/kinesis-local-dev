### Kinesis Producer-Consumer Local Dev and Testing
This app sets up a local kinesis instance using [localstack](https://github.com/localstack/localstack). It also sets up 
local instances of DynamoDB and Cloudwatch. 

The `KinesisInputDStream` builder does not give you the option to point to a local instance of DynamoDB.
The Shard Reader is hard-coded to update aws's prod instance. To get around this problem we've setup a 
reverse proxy to capture outgoing calls to prod server and redirect them to a local DynamoDB instance. We do the same for
Cloudwatch
 
### Local Development
Update hosts file to redirect outgoing aws requests to localstack instances
* Add the following entry to your `/etc/hosts` file
```
127.0.0.1 dynamodb.us-east-1.amazonaws.com
127.0.0.1 monitoring.us-east-1.amazonaws.com
```

Start up localstack and nginx containers
* `docker-compose up -d`

Both the producer and consumer use the [Default Credential Provider Chain](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html)
Ensure one of the credentials is provided. This is required by the Kinesis client library and Kinesis producer library.
Localstack does not perform any authentication

## Appendix

### Generating self signed certs

`openssl req -subj '/CN=localhost' -x509 -newkey rsa:4096 -nodes -keyout key.pem -out cert.pem -days 365`

### Redirecting outbound http request
modify `/etc/hosts`

Add entry `127.0.0.0 <outgoing_url>` to redirect traffic destined to aws dynamoDB and Cloudwatch

### Challenges
* Cannot configure the dynamoDB url, therefore need to setup reverse proxy using nginx and redirecting
traffic destined to to local gateway `127.0.0.1` [Kinesis Stream Issue](https://github.com/localstack/localstack/issues/677)

### Reading
* https://www.thepolyglotdeveloper.com/2017/03/nginx-reverse-proxy-containerized-docker-applications/
* https://dev.to/domysee/setting-up-a-reverse-proxy-with-nginx-and-docker-compose-29jg 