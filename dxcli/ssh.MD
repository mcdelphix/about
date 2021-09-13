## How to embed Delphix CLI command in a bash script

You can manage the Delphix virtualization engine using the command line interface (CLI). To connect to it, you will need to use any SSH client from your machine. Enter the username followed by either the name or IP address (e.g. 10.10.10.1) of the Delphix engine. An example is as follow:
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

------------------------

Once you have that setup, you can embed the commands in bash script file. Create a bash script file (./run.sh) similar to the following:
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

You will then create a launcher script (./runcmd.sh) with the following lines:
```
#!/bin/bash

ssh -tt admin@10.0.1.10 >./runcmd.log 2>./runcmd.err <runcmd.txt
```

When you run `sh ./runcmd.sh`, it will list the current user and output the result to `./runcmd.log` and any errors in  `./runcmd.err`.

--------------------------------

To modify the GUI Idle timeout setting of any user, you can create the following script (set_timeout.sh):
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

To set the timeout for the user admin to 900 seconds, enter the following
```
./set_timeout.sh admin 900
```

Reference:  
[Delphix KB on modifying the GUI idle timeout](https://support.delphix.com/Archived_Articles/Modifying_the_GUI_Idle_Timeout_Setting_(KBA1575))
