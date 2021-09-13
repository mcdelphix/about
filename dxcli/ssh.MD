## How to embed Delphix CLI command in a bash script

You can manage the Delphix virtualization engine using the command line interface (CLI). To connect to it, you will need to use any SSH client from your machine. Enter the username followed by either the name or IP address of the Delphix engine. An example is as follow:
```
ssh delphix@10.10.10.1
```

For a passwordless access to the engine, you can generate a keypair using the ssh-keygen and associate the public key to the Delphix user. Once you have set it up, you can then connect to the engine using the following command:
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
