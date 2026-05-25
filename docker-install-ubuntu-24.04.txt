Docker install ubuntu-24.04
===================================
remove existing docker package
#apt remove docker docker-engine docker.io containerd runc


1.Update the Package Repository
apt update


2: Install Prerequisite Packages


#apt install apt-transport-https ca-certificates curl software-properties-common lsb-release -y

3.Add Docker official GPG key

#sudo install -m 0755 -d /etc/apt/keyrings
 
#curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

#sudo chmod a+r /etc/apt/keyrings/docker.gpg

4: Add Docker Repository
 
#echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 
5: Specify Installation Source
 
#apt-cache policy docker-ce

Update package index
sudo apt update 
 
6: Install Docker
 
 
#apt install docker-ce -y
#apt install docker-ce docker-ce-cli containerd.io –y
#apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  
7: Check Docker Status
  
#systemctl status docker
sudo systemctl enable docker
sudo systemctl start docker

8.Verify Docker installation
docker --version
containerd --version

docker info
