# host 1
IP: 192.168.56.10

vagrant init ubuntu/bionic64
vagrant halt
IP address assign and 
vagrant up
vagrant ssh
sudo -i

# Docker client installing
sudo apt update
sudo apt install -y docker.io

# create a separate docker bridge network 
sudo docker network create --subnet 172.18.0.0/16 vxlan-net
4c6851bb4479df47b6a86943e2ac044fafca1e6483a5b04d0ec37f37d1321806

# list all networks in docke
docker network ls

output
NETWORK ID     NAME        DRIVER    SCOPE
a8fbc7ae74f4   bridge      bridge    local
231c6b346afa   host        host      local
beb0ae23e7fc   none        null      local
1e387e462b61   vxlan-net   bridge    local



sudo docker run -d --net vxlan-net --ip 172.18.0.11 ubuntu sleep 3000

docker ps



## check ip address
docker inspect 1f2f49dc3279 | grep IPAddress



Access docker container
docker exec -it 1f2f49dc3279 bash

# Now we are inside running container
# update the package and install net-tools and ping tools

apt update
apt install net-tools
apt install iputils-ping

Now type exit to come out from the container.

exit


# creaing VxLan Tunnel

# check the bridges list on the hosts
brctl show

# gitbash output

# create a vxlan
# 'vxlan-demo' is the name of the interface, type should be vxlan
# VNI ID is 100
# dstport should be 4789 which a udp standard port for vxlan communication
# 192.168.56.20 is the ip of another host
sudo ip link add vxlan-demo type vxlan id 100 remote 192.168.56.20 dstport 4789 dev enp0s8


# check interface list if the vxlan interface created
ip a | grep vxlan

# make the interface up
sudo ip link set vxlan-demo up

# now attach the newly created vxlan interface to the docker bridge we created
brctl addif br-4c6851bb4479 vxlan-demo


route -n


# enter the running container using exec 
sudo docker exec -it 3382100d3553 bash

# ping the other container IP
ping 172.18.0.21 -c 2



# host 2

IP: 192.168.56.20
vagrant init ubuntu/bionic64
vagrant halt
IP address assign and 
vagrant up
vagrant ssh
sudo -i

# Docker client installing

sudo apt update
sudo apt install -y docker.io


# create a separate docker bridge network 
sudo docker network create --subnet 172.18.0.0/16 vxlan-net
1e387e462b61a729b7efcaee868e4524e8f214bccc62e054c892eba4436b4aa7

# list all networks in docke
docker network ls

output
NETWORK ID     NAME        DRIVER    SCOPE
071a9a3e7397   bridge      bridge    local
c73becfed0f2   host        host      local
f07aba40f963   none        null      local
4c6851bb4479   vxlan-net   bridge    local


sudo docker run -d --net vxlan-net --ip 172.18.0.20 ubuntu sleep 3000

1f2f49dc3279de11dc547fc78fb8406f06312fd0fb75a6d1bf0280ff89590886


docker ps


Access docker container

docker exec -it ec18a034496d bash

# Now we are inside running container
# update the package and install net-tools and ping tools

apt update
apt install net-tools
apt install iputils-ping

Now type exit to come out from the container.
exit
creating VxLan Tunnel


# check the bridges list on the hosts
brctl show

# gitbash output



# create a vxlan
# 'vxlan-demo' is the name of the interface, type should be vxlan
# VNI ID is 100
# dstport should be 4789 which a udp standard port for vxlan communication
# 192.168.56.10 is the ip of another host
sudo ip link add vxlan-demo type vxlan id 100 remote 192.168.56.10 dstport 4789 dev enp0s8

# check interface list if the vxlan interface created
ip a | grep vxlan

sudo ip link set vxlan-demo up

# now attach the newly created vxlan interface to the docker bridge we created
brctl addif br-1e387e462b61 vxlan-demo

# check the route to ensure everything is okay. here '172.18.0.0' part is our concern part.
route -n


# enter the running container using exec 
sudo docker exec -it 050a73bc14e1 bash

# ping the other container IP
ping 172.18.0.11 -c 2


