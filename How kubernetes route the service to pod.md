# How kubernetes route the service to pod
Kubernetes have kube proxy installed on every node.Kube-proxy create IP table rules for every node. where it create entry of service ip and route to IP of pod. 
<img width="1919" height="1081" alt="Screenshot 2025-11-01 095517" src="https://github.com/user-attachments/assets/6ef95150-3099-424c-a371-d0d538f78fec" />
## Cilium Fuctionality
### 1. Cilium In networking
<img width="1914" height="1011" alt="image" src="https://github.com/user-attachments/assets/a6968791-fc42-42fc-bdd7-e6af1ba658b0" />
### 2. Cilium in security
Its work on **L3(IP Adress)** and **L4(Port)** on layer
<img width="1919" height="982" alt="image" src="https://github.com/user-attachments/assets/e1591300-3afb-49c2-b9d6-5e2636f659dd" />
### 3. Culium in obervability
<img width="1916" height="931" alt="image" src="https://github.com/user-attachments/assets/864e5b57-32a4-4739-9bf2-ec2a3f781e11" />
### 4. What is ebpf 
<img width="1910" height="955" alt="image" src="https://github.com/user-attachments/assets/b5d28d1f-52a0-4b87-a453-ddf48e837f30" />
