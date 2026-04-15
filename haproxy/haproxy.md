Configurar un haproxy como balanceador 
-

Run the following command as the default admin user from the bastion host to connect as the jammer user to the load balancer host:

```bash
export SSH_OPTIONS="-o StrictHostKeyChecking=no"
ssh -t $SSH_OPTIONS jammer@aiopslb
```

Next, run the following command as the jammer user on the load balancer host to change to the root user:

```bash
sudo -i

```

Next, run the following command as the root user on the load balancer host to install HAProxy:

```bash
dnf install -y haproxy

```

Next, run the following commands as the root user on the load balancer host to update the HAProxy configuration file, enable HAProxy, and start the HAProxy service:


```bash
(cat > /etc/haproxy/haproxy.cfg << EOF
global
  log         127.0.0.1 local2
  chroot      /var/lib/haproxy
  pidfile     /var/run/haproxy.pid
  maxconn     4000
  user        haproxy
  group       haproxy
  daemon
  stats socket /var/lib/haproxy/stats

defaults
  mode                    http
  log                     global
  option                  httplog
  option                  dontlognull
  option http-server-close
  option forwardfor       except 127.0.0.0/8
  option                  redispatch
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout connect         10s
  timeout client          1m
  timeout server          1m
  timeout http-keep-alive 10s
  timeout check           10s
  maxconn                 3000

frontend aiops-frontend-plaintext
  bind *:80
  mode tcp
  option tcplog
  default_backend aiops-backend-plaintext

frontend aiops-frontend
  bind *:443
  mode tcp
  option tcplog
  default_backend aiops-backend

frontend k3s-frontend
  bind *:6443
  mode tcp
  option tcplog
  default_backend k3s-backend

backend aiops-backend
  mode tcp
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s
  server server0 aiopsc1:443 check
  server server1 aiopsc2:443 check
  server server2 aiopsc3:443 check

backend k3s-backend
  mode tcp
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s
  server server0 aiopsc1:6443 check
  server server1 aiopsc2:6443 check
  server server2 aiopsc3:6443 check

backend aiops-backend-plaintext
  mode tcp
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s
  server server0 aiopsc1:80 check
  server server1 aiopsc2:80 check
  server server2 aiopsc3:80 check
EOF
) && haproxy -c -f /etc/haproxy/haproxy.cfg && systemctl enable haproxy && systemctl start haproxy

```

Finally, run the following command as the root user on the load balancer host to validate your configuration by checking the status of the HAProxy service:
```bash
systemctl status haproxy
```







