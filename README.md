# 🚀 SysAdmin to Cloud Engineer: Infrastructure Runbooks

![Laboratorio de infraestuctura](/assets/img/Code_Generated_Image.png)


Este repositorio consolida mis prácticas y procedimientos (SOPs) a lo largo del roadmap hacia la administración avanzada de sistemas y operaciones Cloud/DevOps. Todo el entorno está construido en un laboratorio propio.

## 🗺️ Plan de Operaciones y Despliegue

| Fase | Áreas Core | Objetivo Técnico (Proyecto) |
| :--- | :--- | :--- |
| **[0. Infraestructura Base](runbooks/0-Infraestructura%20Base/00-proxmox-nested-hyperv.md)** | Hyper-V, Virtualización Anidada, Proxmox VE. | Aprovisionamiento de hipervisor Tipo 1 virtualizado y passthrough de extensiones CPU. |
| **[1. Fundamentos](runbooks/1-Fundamentos/01-ubuntu-deployment-rbac.md)** | Terminal, permisos, FHS, instalación de Ubuntu Server LTS. | Aprovisionamiento de Ubuntu Server en VM, gestión de RBAC y particionado tradicional. |
| **2. Storage & Servicios** | LVM, control de procesos, `systemd`, hardening SSH, ufw/nftables. | Despliegue de stack web seguro (Firewall + SSH keys). Simulación y resolución de Kernel Panic/Boot failure. |
| **3. Automatización** | Bash scripting (awk, sed, grep), crontab, fundamentos de Ansible. | Automatización de backups mediante scripts y primer playbook de configuración de estado deseado (Ansible). |
| **4. Seguridad & Logs** | SELinux, AppArmor, auditoría de RHEL, análisis centralizado de logs. | Hardening integral (SSH, SELinux estricto, fail2ban, auditd). |
| **5. Infraestructura como Código (IaC)** | Terraform (módulos, state management), integración K8s. | Aprovisionamiento completo (VPC, Compute, Security Groups) combinando Terraform y Ansible. |
| **6. Contenedores y Orquestación** | Docker, K8s, Helm, GitOps (ArgoCD/Flux). | Despliegue de clúster K3s local, despliegues vía Helm y sincronización de estado con ArgoCD. |
| **7. Alta Disponibilidad** | Proxmox (clustering, HA), storage distribuido, CI/CD pipelines. | Clúster Proxmox multimodo, pipelines CI/CD y stack de observabilidad (Prometheus + Grafana). |

## 📚 Bibliografía y Referencias Clave
- **TechWorld with Nana:** IaC y Contenedores (Docker, Ansible, Terraform, Kubernetes).
- **LearnLinuxTV:** Core de Linux, systemd y hardening básico.
- **Jeff Geerling:** Casos de uso avanzados en Ansible y K8s.
- **KodeKloud:** Hands-on labs de infraestructura (Kubernetes, Terraform, Linux).
- **Craft Computing:** Arquitectura de virtualización sobre Proxmox.
