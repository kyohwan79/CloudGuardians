## clone하고 sudo apt update 중 에러 발생 시

```bash
sudo rm -rf /var/lib/apt/lists/partial
sudo mkdir -p /var/lib/apt/lists/partial
sudo apt-get clean
sudo apt-get update

```
