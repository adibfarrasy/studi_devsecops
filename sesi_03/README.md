# SSH and transfer files from host to VM
This is the continuation of the sesi_02 homework.

## Pre-requisites
- VM from sesi_02 set up and running

## SSH Steps
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
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "<your-public-key-content>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

nano /etc/ssh/sshd_config
# Uncomment lines containing 'PubkeyAuthentication' and 'AuthorizedKeysFile'

systemctl restart ssh
```

### 5. Connect from host machine to VM
```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 localhost
```

## Transfer Files via `scp` Command
### 1. Create file and folder to be sent to the VM
```bash
mkdir scp_test
cd scp_test
touch hello.txt
cat << EOF > hello.txt
hello world!
EOF
```

### 2. Transfer the folder to the VM
```bash
scp -r -P 2222 -i ~/.ssh/id_ed25519 /path/to/scp_test adibf@localhost:/home/adibf
```

### 3. Verify that the folder is transferred
```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 localhost
ls scp_test
# Output
# scp_test
```

## Transfer Files via `sftp` Command
### 1. Create file and folder to be sent to the VM
```bash
mkdir sftp_test
cd sftp_test
touch hello.txt
cat << EOF > hello.txt
hello world!
EOF
```

### 2. Transfer the folder to the VM
```bash
sftp -P 2222 -i ~/.ssh/id_ed25519 adibf@localhost
sftp> ls
sftp> mkdir sftp_test
sftp> put -r /path/to/sftp_test/* /home/adibf/sftp_test/
sftp> bye
```

### 3. Verify that the folder is transferred
```bash
ssh -i ~/.ssh/id_ed25519 -p 2222 localhost
ls sftp_test
# Output
# sftp_test
```
