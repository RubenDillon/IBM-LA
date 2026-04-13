# Trabajando con un bastion
=

En este archivo subo algunas mejores practicas para cuando es necesario trabajar con un bastion para acceder a una red segura

Permitir acceso root en los nodos
-

```bash
export SSH_OPTIONS="-o StrictHostKeyChecking=no"
for host in host1 host2 host3 host4; do
    ssh -t $SSH_OPTIONS root@$host 'sudo bash -s' << 'EOF'
        echo "Updating sshd_config on $(hostname)..."
        sed -i 's/^PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config

        echo "Restarting sshd on $(hostname)..."
        systemctl restart sshd

        echo "PermitRootLogin set on $(hostname)"
EOF
done
```

Run the following command as the default admin user on the bastion host to store the local public SSH key on bastion in the environment variable PUBLIC_KEY and then add it to the root user's authorized_keys file on host1. This will allow connection between bastion and host1 without having to enter a password.

```bash
PUBLIC_KEY=$(cat ~/.ssh/id_rsa.pub)
ssh root@host1 "sudo -i bash -c 'echo \"$PUBLIC_KEY\" >> ~/.ssh/authorized_keys' && echo \"SSH key added to aiopsc1 successfully\""

```

Run the following command as the default admin user on the bastion host to create a new SSH key that will be used for the hosts:

```bash
ssh-keygen -t rsa -f ~/.ssh/hosts -N "" -C "hosts Key"
```



Run the following command as the default admin user on the bastion host to store the new public SSH key in the environment variable PUBLIC_KEY and then add it to the root user's authorized_keys file on all the hosts except for host1.


```bash

PUBLIC_KEY=$(cat ~/.ssh/aiops-cluster.pub)
for i in host2 host3 host4; do
  ssh root@$i "sudo -i bash -c 'mkdir -p ~/.ssh && chmod 700 ~/.ssh && echo \"$PUBLIC_KEY\" >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys' && echo \"SSH key added to $i successfully\""
done

```

Finally, run the following commands as the default admin user on the bastion host set up the root user on host1 to use the new SSH private key to connect to the other nodes:

```bash

scp ~/.ssh/hosts root@host1:/tmp/aiops_key && echo "Key copied successfully" && \
ssh root@host1 "sudo -i bash -c 'mkdir -p ~/.ssh && chmod 700 ~/.ssh && \
mv /tmp/aiops_key ~/.ssh/hosts && chmod 600 ~/.ssh/hosts && chown root:root ~/.ssh/hosts && \
echo \"Host host2 host3 host4 192.168.252.11 192.168.252.12 192.168.252.13 192.168.252.14\" > ~/.ssh/config && \
echo \"    User root\" >> ~/.ssh/config && \
echo \"    IdentityFile ~/.ssh/host\" >> ~/.ssh/config && \
echo \"    StrictHostKeyChecking no\" >> ~/.ssh/config && \
chmod 600 ~/.ssh/config && echo \"All operations completed successfully\"'"

```



Fuente
- documentacion para desplegar AIOPs desde un bastion (https://ibm.github.io/waiops-tech-jam/labs/cloud-pak-aiops/install-lab-linux/installation/)
