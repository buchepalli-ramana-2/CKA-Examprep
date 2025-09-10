
# CKA Practice Environment on Azure using Terraform

This project sets up a Kubernetes practice environment for the Certified Kubernetes Administrator (CKA) exam using Azure resources.

## 📦 Project Structure
- `main.tf`: Defines networking resources (VNet, subnets, NICs, public IP)
- `vm.tf`: Defines virtual machines (Windows client, Ubuntu control plane and worker nodes)
- `outputs.tf`: Outputs IP addresses for easy access

---

## 🚀 Deployment Steps

### 1. Initialize Terraform
```bash
terraform init
```

### 2. Review the Execution Plan
```bash
terraform plan
```

### 3. Apply the Configuration
```bash
terraform apply
```
Confirm with `yes` when prompted.

---

## 🔧 Kubernetes Provisioning (Manual Setup)
SSH into each Ubuntu VM and run the following:

### Control Plane Node
```bash
sudo apt update && sudo apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl containerd
sudo kubeadm init
```

### Worker Nodes
```bash
sudo apt update && sudo apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl containerd
```
Join the cluster using the token from the control plane node.

---

## 🔐 Optional: Integrate Azure Bastion
To securely access VMs without public IPs:
```hcl
resource "azurerm_bastion_host" "bastion" {
  name                = "cka-bastion"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_name            = "cka-bastion"
  ip_configuration {
    name                 = "configuration"
    subnet_id            = azurerm_subnet.bastion_subnet.id
    public_ip_address_id = azurerm_public_ip.bastion_public_ip.id
  }
}
```

## 🌐 Optional: NAT Gateway for Internet Access
To allow private VMs to access the internet:
```hcl
resource "azurerm_nat_gateway" "nat" {
  name                = "cka-nat"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  sku_name            = "Standard"
  public_ip_ids       = [azurerm_public_ip.nat_public_ip.id]
}
```
Associate it with the cluster subnet.

---

## 📘 Notes
- Change default passwords before production use.
- Use SSH keys for secure access.
- Monitor costs in Azure Portal.
