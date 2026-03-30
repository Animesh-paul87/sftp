# sftp configuration 1
==============================================
1.  Create user with No shell access
```
sudo useradd -m -s /sbin/nologin test1
sudo passwd test1
```
2. Setup the chroot directory (Must be owned by root)
```
sudo chown root:root /home/test1
sudo chmod 755 /home/test1
```
3. Crete a 'uploads' folder where the user can actually write files
```
sudo mkdir /home/test1/uploads
sudo chown test1:test1 /home/test1/uploads
```
Configure SSH Daemon
=====================
we need to tell the ssh service to treat test1 different by using the internal-sftp subsystem.
edit /etc/ssh/sshd_config
----------------------------
```
Match User test1
ChrootDirectory /home/test1
#Force SFTP and apply a umask for proper file Permission
ForceCommand internal-sftp 
AllowTcpForwarding no
X11Forwarding no
PermitTunnel no
AllowAgentForwarding no
```
Verify and restart the ssh service
-----------------------------------------
To check the Configuration
--------------------------
```
sshd -t
```
restart the sshd service 
---------------------------
```
systemctl restart sshd
```





# sftp configuration Using bind Mount 2 (configuration need to check not working)
==============================================
# Create a Directory Structure (will Create a 'Jail' Folder structure that Root Owns, and a "Home" folder that test1 ownes, we need to map them together.

1.  Create user with No shell access
```
sudo useradd -m -s /sbin/nologin test1
```
# Crete the sftp jail (Must be owned by root)
 ```
 sudo mkdir -p /sftp/test1
 sudo chown root:root /sftp/test1
 sudo chown 755 /sftp/test1
```
 2. The actual storage location (Owned by test1), This is where the actual file will physically stay
```
sudo mkdir -p /home/test1_files
sudo chown test1:test1 /home/test1_files
```
3. The "Mount Bind" ----> below command makes the system think /home/test1_files is /sftp/test1, This satisfies the root requirement of the jail
4. while giving the user write access to the "root" of their login
```
sudo mount --bind  /home/test1_files /sftp/test1
```
To make this permanent after a reboot, add this line to fstab
```
fstab entry
====================
 /home/test1_files       /sftp/test1    none bind 0 0
```
sshd_config
```
Match User test1
#User is locked inside their own folder name only
ChrootDirectory /sftp/test1
#Force SFTP and apply a umask for proper file Permission
ForceCommand internal-sftp
AllowTcpForwarding no
X11Forwarding no
PermitTunnel no
AllowAgentForwarding no
```






# sftp configuration 3
=======================

# why this configuration is differ from the normal configuration
# when the user want to store file inside their home folder they dont want the 3 level folder structure.

System Preperation
=============================================================================
1. Create user with No shell access
```
sudo useradd -m -s /sbin/nologin test1
```
2. set the directory ownership to root (Required for chroot)
```
sudo chown root:root /home/test1
sudo chmod 755 /home/test1
```
3. Apply Access control lists (ACL)
# This allows 'test1' to write to a root-owned folder
```
setfacl -m u:test1:rwx /home/test1
```
4. set the default ACL's
# Below Line ensure that every new file uploaded is owned/writable by 'test1'
```
setfacl -d -m u:test1:rwx /home/test1
```
SSHD_CONFIG
==============================================================================
```
Match User test1
#User is locked inside their own folder name only
ChrootDirectory /home/%u
#Force SFTP and apply a umask for proper file Permission
ForceCommand internal-sftp -u 0002
AllowTcpForwarding no
X11Forwarding no
PermitTunnel no
AllowAgentForwarding no
```

Apply Changes
==================================
# Test the configuration is OK
```
sshd -t
```
# Restart the sshd service to make changes
sudo systemctl restart sshd
