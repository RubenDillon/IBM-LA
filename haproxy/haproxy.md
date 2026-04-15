Configurar un haproxy como balanceador 
-

Run the following command as the default admin user from the bastion host to connect as the jammer user to the load balancer host:

```bash
export SSH_OPTIONS="-o StrictHostKeyChecking=no"
ssh -t $SSH_OPTIONS jammer@aiopslb
```
