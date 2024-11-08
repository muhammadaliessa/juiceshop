![image](https://github.com/user-attachments/assets/ad1deebd-4f9f-429e-b403-a61ebc806806)

0.	use vagrant as IaC to provision two machines  over Virtual box.

1.	RKE2 master 
1.1.	mkdir -p /etc/rancher/rke2/
1.2.	curl -sfL https://get.rke2.io | sh –
1.3.	systemctl enable rke2-server.service
1.4.	systemctl start rke2-server.service
1.5.	cp /etc/rancher/rke2/rke2.yaml  ~/.kube/config
1.6.	kubetctl get all

2.	RKE2 worker (agent node)
2.1.	 mkdir -p /etc/rancher/rke2/
2.2.	curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh –
2.3.	systemctl enable rke2-agent.service
2.4.	get the tocken from master node >> cat /var/lib/rancher/rke2/server/node-token
2.5.	insert it in config.yaml in worker.
cat <<EOF | sudo tee /etc/rancher/rke2/config.yaml
token: K10afe90efc656de75974cbfecdd97cd0f3fb22fe6679b4ab542abcf4d3cb712eec::server:aa43b45ba653f0b4f29167777a2f2fa7
server:
  - https://k8s.local:9345
EOF
2.6.	systemctl start rke2-agent.service

3.	validation 
 
![image](https://github.com/user-attachments/assets/96447930-d26d-4fdd-935b-6127f1280ce1)



4.	troubleshooting 
•	Systemctl status rke2-server.service
•	Systemctl status rke2-agent.service
•	journalctl -u rke2-server.service -f

5.	Restrict apiserver from specific IP through Iptables
restict access for speicf ip
sudo iptables -A INPUT -p tcp -s 192.168.2.100 --dport 6443 -j ACCEPT
sudo iptables -A INPUT -p tcp -s 192.168.2.77 --dport 6443 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 6443 -j DROP
iptables -L -v -n | grep 6443	
 
6.	Create namespace for project
Kubectl create ns project
 
7.	Deploy nginx-ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

8.	Deploy juice-shop app     
kubectl create deployment juice-shop  --image=docker.io/bkimminich/juice-shop --replicas=2 -n project

9.	Expose app internally 
kubectl expose deployment hr-web-app --name juice-service  --port 3000 -n project

 






10.	Expose app externally using ingress
mkdir /server
touch juice-shop-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: juice-shop-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  namespace: project
spec:
  rules:
  - host: juice.shop
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: juice-shop-service
            port:
              number: 443

 
11.	Validate app is accessible through browser
 

 
