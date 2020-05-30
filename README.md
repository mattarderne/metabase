# Metabase

## Requirements

* AWS account, ability to setup EC2 and RDS instances
* Docker-machine installed

## Setup

1. Create an AWS postgreSQL RDS instance  (t2.micro should be fine, this is just the metadata)
1. Create a `.env` file for your RDS DB password
```
vi .env
RDS_CLIENT_ID=<RDS_USERNAME_READ_AND_WRITE>
RDS_CLIENT_PWD=<VERY_SECURE_PASSWORD>
```
1. Open docker-compose.yml and replace `MB_DB_HOST` variable with your RDS instance
1. Open [default.conf](/nginx/default.conf) and change the variables as appropriate. You can setup an elastic IP or domain name here.
1. Get some certs from your Devops team, make a `certs` folder in this repo and copy them in
```
mkdir nginx/certs
cp ~/certs/<cert-name>.crt nginx/certs
cp ~/certs/<cert-name>.key nginx/certs
```

## Deploy
1. Run the following docker-machine, with the variables replaced with your region/vpc/subnet/key
```
docker-machine create \
  --amazonec2-vpc-id <vpc_id> \
  --amazonec2-region <aws_preferred_region>  \
  --amazonec2-instance-type t2.medium \
  --amazonec2-use-private-address \
  --amazonec2-keypair-name <your_AWS_keypair_name> \
  --amazonec2-subnet-id <subnet> \
  --amazonec2-ssh-keypath ~/<directory_of_your_keypath>.pem \
  --driver amazonec2 \
  data-metabase
```
1. Run the following to get the machine IP, for later
```
docker-machine ip data-metabase
```
1. Connect to the now running Docker Machine with the following 
```
eval $(docker-machine env data-metabase)
```
Only need to run this once, test it worked by running `docker ps`
1. Build & Start Metabase docker container
```
eval $(docker-machine env data-metabase)
docker-compose build
docker-compose up
```
Your Metabase should now be available at the IP or domain you used!

## Maintain

* Stop Metabase docker container
```
eval $(docker-machine env data-metabase)
docker ps
docker stop <container_id>
```
* Stop EC2 server
```
docker-machine stop data-metabase
```
* Start EC2 server
```
docker-machine start data-metabase
```

* Upgrade Metabase
```
eval $(docker-machine env data-metabase)
docker ps
docker stop <container_id>
docker pull metabase/metabase:latest
docker-compose build
docker-compose up
```

## Devops Notes

* Use an Elastic IP, add to the whitelisted list on Snowflake IP policy
* To get server logs:
```
eval $(dm env data-metabase)
docker ps
# docker logs -f 08b75f98c9a7   #all logs
docker logs --tail 100 08b75f98c
# may need to change 08b75f98c9a7