
# DevOps Interview Task Answers

This document provides answers and implementations for common DevOps interview tasks involving Terraform, shell scripting, and AWS CloudWatch with Python.

---

## 1. Design and implement a Terraform module for a multi-tier application infrastructure with proper state management

### 🔧 Requirements

- Azure CLI authenticated
- Terraform initialized with remote backend

### 🗂 Directory Structure

```
terraform/
├── backend.tf
├── main.tf
├── outputs.tf
├── variables.tf
└── modules/
    └── multi_tier/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### 📄 `backend.tf`

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "multi-tier-app.tfstate"
  }
}
```

### 📄 Module (`modules/multi_tier/main.tf`)

```hcl
resource "azurerm_virtual_network" "vnet" {
  name                = "vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = var.resource_group_name
}

resource "azurerm_subnet" "web" {
  ...
}

resource "azurerm_subnet" "app" {
  ...
}

resource "azurerm_network_interface" "web_nic" {
  ...
}

resource "azurerm_linux_virtual_machine" "web_vm" {
  ...
}
```

### 📄 Module Variables (`modules/multi_tier/variables.tf`)

```hcl
variable "location" {}
variable "resource_group_name" {}
```

### 📄 Usage (`main.tf`)

```hcl
provider "azurerm" {
  features {}
}

module "multi_tier" {
  source              = "./modules/multi_tier"
  location            = "East US"
  resource_group_name = "app-rg"
}
```

---

## 2. Write a shell script to automate log rotation and system cleanup with error handling and notifications

```bash
#!/bin/bash

LOG_DIR="/var/log/myapp"
BACKUP_DIR="/var/log/myapp/backup"
DAYS_TO_KEEP=7
EMAIL="admin@example.com"
HOST=$(hostname)

mkdir -p "$BACKUP_DIR"

for file in "$LOG_DIR"/*.log; do
  if [ -f "$file" ]; then
    cp "$file" "$BACKUP_DIR/$(basename $file).$(date +%F).gz"
    gzip "$BACKUP_DIR/$(basename $file).$(date +%F)"
    cat /dev/null > "$file"
  fi
done

find "$BACKUP_DIR" -type f -name "*.gz" -mtime +$DAYS_TO_KEEP -exec rm {} \;

if [ $? -ne 0 ]; then
  echo "Log rotation or cleanup failed on $HOST" | mail -s "Log Rotation Error on $HOST" $EMAIL
  exit 1
else
  echo "Log rotation and cleanup completed successfully on $HOST" | mail -s "Log Rotation Success" $EMAIL
fi
```

---

## 3. Create a Python script for parsing and analyzing AWS CloudWatch metrics with data visualization

### 📦 Dependencies

```bash
pip install boto3 pandas matplotlib
```

### 🐍 Python Script

```python
import boto3
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

cloudwatch = boto3.client('cloudwatch')

def get_metrics(namespace, metric_name, instance_id):
    end_time = datetime.utcnow()
    start_time = end_time - timedelta(hours=1)

    response = cloudwatch.get_metric_statistics(
        Namespace=namespace,
        MetricName=metric_name,
        Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
        StartTime=start_time,
        EndTime=end_time,
        Period=300,
        Statistics=['Average']
    )

    datapoints = response['Datapoints']
    df = pd.DataFrame(datapoints)
    if df.empty:
        print("No data available")
        return

    df = df.sort_values('Timestamp')
    plt.plot(df['Timestamp'], df['Average'])
    plt.title(f"{metric_name} for {instance_id}")
    plt.xlabel("Time")
    plt.ylabel("Average")
    plt.grid()
    plt.tight_layout()
    plt.show()

# Example usage
get_metrics("AWS/EC2", "CPUUtilization", "i-1234567890abcdef0")
```

---

## ✅ Optional Enhancements

- Add CI/CD pipeline for infrastructure deployments
- Integrate with Grafana for richer dashboards
- Use Azure Monitor or AWS CLI for additional insights

---
