# Deploy a Flask web app on an AWS Ubuntu server

Step by step instruction.

## Create an Instance

1. Login in to your AWS console and go the EC2 Dashboard.
2. Head over to Launch Instance and choose for Ubuntu Server.
3. You can leave all default settings and press on Review and Launch.
4. Create a Key Pair, download the Key Pair file and store it locally on
`~/.ssh/example_key.pem`

## Connect to the Instance by SSH

Open a terminal on your local computer.

Your key must not be publicly viewable for SSH to work. Use this command if
needed:

```bash
$ chmod 400 ~/.ssh/example_key.pem
```


```bash
$ ssh -i "~/.ssh/example_key.pem" ubuntu@ec0-00-00-000-000.eu-central-1.compute.amazonaws.com
```

Change to string `ubuntu@ec0-00-00-000-000.eu-central-1.compute.amazonaws.com`
the IP address of your Instance. You can find this IP address under Public DNS
(IPv4) at the EC2 Dashboard on the Instance tab.

## Setting up the Instance
