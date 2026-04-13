Trabajando con un bastion
-

En este archivo subo algunas mejores practicas para cuando es necesario trabajar con un bastion para acceder a una red segura

Permitir acceso root en los nodos
-

´´´
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
´´´
