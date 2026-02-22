

```shell
wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | sudo apt-key add -
sudo add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/
sudo apt update
sudo apt install -y adoptopenjdk-8-hotspot
echo "JAVA_HOME=/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64" >> ~/.bashrc
```
