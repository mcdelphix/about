## How to embed Delphix CLI command in a bash script

You can manage the Delphix virtualization engine using the command line interface (CLI). To connect to it, you will need to use any SSH client from your machine. Enter the username followed by either the name or IP address (e.g. 10.10.10.1) of the Delphix engine, you will be prompted for a password. An example is as follow:
```
ssh delphix@10.10.10.1
```

For a passwordless access to the engine, you can generate a new authentication key pairs (public/private) using the ssh-keygen and associate the public key to the Delphix user. To set it up, enter the following:
```
ip-10-10-10-1> user current
ip-10-10-10-1 user "delphix"> update
ip-10-10-10-1 user "delphix" update *> set publicKey="[CONTENT OF THE PUBLIC KEY=/home/delphix/internal/demo-scripts/dxkey.pub]"
ip-10-10-10-1 user "delphix" update *> commit
```

Once you have set it up, you can then connect to the engine using the following command:
```
ssh -i /home/delphix/internal/demo-scripts/dxkey delphix@10.10.10.1
```
Or, if you have a `config` file in the `~/.ssh` directory with the following lines:
```
Host *
  stricthostkeychecking=no
  forwardX11=no
  UserKnownHostsFile=/dev/null
  IdentityFile=/home/delphix/internal/demo-scripts/dxkey
```
you can connect to the engine using the following command:
```
ssh delphix@10.10.10.1
```

The SSH config file `(.ssh/config)` makes it easier for you to connect to the engine by definining the specific settings for each SSH host. In the above example, for every host we are connecting to, we will be using the private key file `/home/delphix/internal/demo-scripts/dxkey`.

------------------------

Once you have that setup, you can embed the commands in bash script file. Create a bash script file (*./run.sh*) similar to the following:
```
#!/bin/bash

ssh admin@10.0.1.10 >./run.log 2>./run.err "$( cat <<'EOT'
ls
user
current
ls
exit
EOT
)"
```

When you run `sh ./run.sh`, it will list the current user and output the result to `./run.log` and any errors in `./run.err` 

----------------

Alternatively, you can place all the Delphix command in a file, say `./runcmd.txt` 
```
ls
user
current
ls
exit
```

You will then create a launcher script (*./runcmd.sh*) with the following lines:
```
#!/bin/bash

ssh -tt admin@10.0.1.10 >./runcmd.log 2>./runcmd.err <runcmd.txt
```

When you run `sh ./runcmd.sh`, it will list the current user and output the result to `./runcmd.log` and any errors in  `./runcmd.err`.

--------------------------------

To modify the GUI Idle timeout setting of any user, you can create the following script (*set_timeout.sh*):
```
#!/bin/bash

[ $# -lt 2 ] && { echo "Usage: $0 Delphix_User Timeout_Sec"; exit 1; }

cmdFile=./input.txt

cat > $cmdFile  <<EOF
select $1
update 
set sessionTimeout=$2
commit
exit
EOF

ssh -tt admin@10.0.1.10 >./run.log 2>./run.err < $cmdFile
```

To set the timeout for the user **admin** to **900** seconds, enter the following
```
./set_timeout.sh admin 900
```


--------------------------------

To change the password of a Delphix user, you can create the following script (*set_password.sh*):
```
#!/bin/bash

[ $# -lt 2 ] && { echo "Usage: $0 Delphix_User NewPassword"; exit 1; }

cmdFile=./input.txt

cat > $cmdFile  <<EOF
select $1
updateCredential
set newCredential.password=$2
commit
exit
EOF

ssh -tt admin@10.0.1.10 >./run.log 2>./run.err < $cmdFile
```

To set the timeout for the user **john** to **900** seconds, enter the following
```
./set_password.sh john s3cr3t99
```

--------------------------------

To add a Windows environment, you can create the following script (*add_winenviron.sh*):
```
#!/bin/bash

[ $# -lt 2 ] && { echo "Usage: $0 Delphix_User NewPassword"; exit 1; }

cmdFile=./input.txt

cat > $cmdFile  <<EOF

/environment
create
set type=HostEnvironmentCreateParameters
set hostEnvironment.type=WindowsHostEnvironment
set hostEnvironment.name=<Source environment name>
set hostEnvironment.proxy=<target host name>
set hostParameters.type=WindowsHostCreateParameters
set hostParameters.host.type=WindowsHost
set hostParameters.host.addresses="<Source host IP address or hostname>"
set primaryUser.name="<domain\username>"
set primaryUser.credential.type=PasswordCredential
set primaryUser.credential.password=<password>
commit
exit
EOF

ssh -tt admin@10.0.1.10 >./run.log 2>./run.err < $cmdFile
```

To set the timeout for the user **admin** to **900** seconds, enter the following
```
./add_winenviron.sh admin s3cr3t99
```

--------------------------------

To add a Linux environment, you can create the following script (*add_lnxenviron.sh*):
```
#!/bin/bash

[ $# -lt 2 ] && { echo "Usage: $0 Delphix_User NewPassword"; exit 1; }

cmdFile=./input.txt

cat > $cmdFile  <<EOF

environment create
set type=HostEnvironmentCreateParameters
set hostEnvironment.type=UnixHostEnvironment
set hostParameters.type=UnixHostCreateParameters
set primaryUser.credential.type=PasswordCredential
set hostParameters.host.addresses="<Source host IP address or hostname>"
set hostParameters.host.toolkit=/var/delphix
set primaryUser.name=oracle
set primaryUser.credential.password=<password>
commit
exit
EOF

ssh -tt admin@10.0.1.10 >./run.log 2>./run.err < $cmdFile
```

To set the timeout for the user **admin** to **900** seconds, enter the following
```
./add_lnxenviron.sh admin s3cr3t99
```

--------------------------------

To list the timeflow , you can create the following script (*list_timeflow.sh*):
```
#!/bin/bash

[ $# -lt 0 ] && { echo "Usage: $0 "; exit 1; }

cmdFile=./input.txt

cat > $cmdFile  <<EOF
timeflow
ls
commit
exit
EOF

ssh -tt admin@10.0.1.10 >./run.log 2>./run.err < $cmdFile
```

To list the timeflows, enter the following
```
./list_timeflow.sh
```

--------------------------------

To take a snapshot, create the following script (*take_snapshot.sh*):
```
#!/bin/bash

[ $# -lt 1 ] && { echo "Usage: $0 VDB_name"; exit 1; }

cmdFile=./input.txt

cat > $cmdFile  <<EOF
database
select $1
sync
commit
/snapshot
ls
exit
EOF

ssh -tt admin@10.0.1.10 >./run.log 2>./run.err < $cmdFile
```

To snapshot the VDB vdb_test, enter the following
```
./take_snapshot.sh vdb_test
```

Reference:  
[Delphix KB on modifying the GUI idle timeout](https://support.delphix.com/Archived_Articles/Modifying_the_GUI_Idle_Timeout_Setting_(KBA1575))
