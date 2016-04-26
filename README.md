## Linux Server Configuration

A baseline installation of a Linux distribution on a virtual machine via Flask Web application.

### Server Details

The catalog web app can be accesed through following links:

** Server IP: ** http://52.37.200.141/
** Host name: ** http://ec2-52-37-200-141.us-west-2.compute.amazonaws.com/

### Launch a virtual machine and SSH into server

1. Launch a  virtual machine using udacity account.
2. Download the private key and notice down the public IP address.
3. Change file rights:  
  `$ chmod 600 ~/.ssh/udacity_key.rsa`
4. SSH into the instance:  
  `$ ssh -i ~/.ssh/udacity_key.rsa root@52.37.200.141`
