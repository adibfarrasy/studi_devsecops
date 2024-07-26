# Installing Ubuntu Server to Qemu VM via cloud-init
Host: any Linux distro (I use NixOS).

## Pre-requisites
- [Download Ubuntu Server 24.04](http://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img)
- [Install Qemu VM](https://www.qemu.org/download/)
- [Install Python3](https://www.python.org/downloads/)

## Install Steps
### 1. Create a directory
In this example, I named mine `cloud-init`
```bash
mkdir cloud-init
cd cloud-init
```
Copy the downloaded Ubuntu image to the created directory.

### 2. Define user data
We need to create 3 files, `user-data`, `meta-data`, and `vendor-data`.
```bash
cat << EOF > user-data
#cloud-config
password: password
chpasswd:
  expire: False

EOF

cat << EOF > meta-data
instance-id: someid/somehostname

EOF

# empty file
touch vendor-data
```

### 3. Start a Python default file server
To send the files in the current directory to the Ubuntu Server.
Open a new terminal and run the command
```bash
python3 -m http.server --directory .
```

### 4. Start Qemu
```bash
qemu-system-x86_64 \
    -net nic \
    -net user,hostfwd=tcp::2222-:22 \
    -machine accel=kvm:tcg \
    -m 2048 \
    -nographic \
    -hda noble-server-cloudimg-amd64.img \
    -smbios type=1,serial=ds='nocloud;s=http://10.0.2.2:8000/'
```

- `-net nic,hostfwd=tcp::2222-:22` enables Network Interface Card (NIC) to qemu.
  Allow port forwarding to SSH from the host port `2222`.
- `-net user` allows user mode access for networking
- `-machine accel=kvm:tcg` enables acceleration method, if available.
- `-m 2048` sets memory to 2GB
- `-nographic` sets command-line mode
- `serial=ds='nocloud;s=http://10.0.2.2:8000/'` configures the cloud-init data 
source to be NoCloud. The `s=` parameter specifies the URL from which the VM will
fetch its cloud-init data. The URL to `http://10.0.2.2:8000/` is an IP address on 
the default QEMU user network, mapped to the host's localhost.

> NOTE: use 'Ctrl-a x' to tear down the server. 

### 5. Login
Once you see `Ubuntu 24.04 LTS ubuntu ttyS0` you should be able to login using
username `ubuntu` and password `password`.

## User SSH Steps
### 1. (On virtual machine) Create a new user with password
```bash
sudo useradd -m adibf
sudo id adibf 
# Output 
# uid=1001(adibf) gid=1001(adibf) groups=1001(adibf)

sudo passwd adibf
# Output
# New password:
# Retype new password:
# passwd: password updated successfully

sudo systemctl start ssh
sudo systemctl enable ssh
```

### 2. (On host machine) Create SSH key
```bash
ssh-keygen -t ed25519
```
The default location is `~/.ssh/id_ed25519`.

### 3. (On host machine) Get public SSH key
```bash
cat ~/.ssh/id_ed25519.pub
```
Copy the output of this command for the next step.

### 4. (On virtual machine) Setup SSH on the VM
```bash
su -
cd /home/adibf
mkdir -p .ssh
chmod 700 .ssh
echo "<your-public-key-content>" >> .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
chown adibf:adibf .ssh
chown adibf:adibf .ssh/authorized_keys

nano /etc/ssh/sshd_config
# Uncomment lines containing 'PubkeyAuthentication' and 'AuthorizedKeysFile'

systemctl restart ssh
```

### 5. Connect from host machine to VM
```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 localhost
```
