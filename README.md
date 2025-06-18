# Docker Learning Lab Repository

A comprehensive collection of Docker learning materials, featuring detailed lab environment setup guides and essential Docker commands for hands-on containerization learning.

## 📚 Repository Contents

### Day 01 - Docker Lab Environment Setup
**Comprehensive Virtual Machine & Docker Installation Guide**

### Day 02 - Docker Commands Reference
Essential Docker commands and usage examples for daily container operations.

## 🛠️ Lab Environment Specifications

### Virtual Machine Configuration
- **Platform**: VMware Workstation 16 Pro
- **OS**: CentOS Stream 9 (64-bit)
- **Resources per VM**: 4 vCPUs, 3GB RAM, 100GB Storage
- **Network**: NAT configuration with static IPs
- **VMs**: docker1 (192.168.2.10), docker2 (192.168.2.20)

### Prerequisites
- VMware Workstation 16 Pro (free from Broadcom)
- CentOS Stream 9 ISO image
- Minimum 8GB RAM on host machine
- 200GB+ available storage space
- Administrative privileges

## 🚀 Quick Start Guide

1. **Follow Day 01 Setup**: Complete the comprehensive environment setup
   - Create and configure virtual machines
   - Install CentOS Stream 9
   - Configure networking and hostnames
   - Install Docker Engine on both VMs

2. **Verify Installation**: 
   ```bash
   # Test Docker installation
   docker container run hello-world
   
   # Test web server (on docker1)
   docker container run -p 80:80 nginx
   ```

3. **Access Lab Environment**: Both VMs configured with:
   - Username: `root`
   - Password: `centos`
   - SSH access enabled

### Common Issues
- **Black screen on VM boot**: Ensure ISO image is properly mounted
- **Network connectivity issues**: Verify NAT configuration and static IP settings
- **Docker permission errors**: Ensure Docker service is running and user has proper permissions
- **VM performance**: Adjust CPU/memory allocation based on host resources

### Useful Commands
```bash
# Check network configuration
ip address
ip route

# Verify Docker status
systemctl status docker

# Test connectivity between VMs
ping docker1.example.com
ping docker2.example.com
```

## 🌐 Additional Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [CentOS Stream Documentation](https://centos.org/stream/)
- [VMware Workstation Documentation](https://docs.vmware.com/en/VMware-Workstation-Pro/)
- [Docker Installation Guide](https://docs.docker.com/engine/install/centos/)

## 📝 Notes

- All guides are tested on VMware Workstation 16 Pro with CentOS Stream 8
- Network configurations assume NAT networking mode
- Setup scripts are sourced from GitHub for automated configuration
- Environment is designed for learning purposes with relaxed security settings

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

*Ready to containerize? Start with Day 01 and build your Docker expertise! 🐳*
