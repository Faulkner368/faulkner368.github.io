# How to Use Raspberry Pi for Self-Hosting GitHub Action Runners: A Comprehensive Guide

GitHub Actions has become an indispensable tool for developers to automate CI/CD pipelines. While GitHub provides hosted runners, self-hosting GitHub Action runners can give you more control, reduce costs, and utilize your own hardware efficiently. In this guide, we’ll explore how to use a **Raspberry Pi** for self-hosting GitHub Action runners, including detailed setup instructions, benefits, and tips to optimize performance.

---

## **Why Use Raspberry Pi for Self-Hosting GitHub Action Runners?**

### **Benefits of Self-Hosting**

1. **Cost Savings**: GitHub-hosted runners charge usage fees after a certain threshold. Self-hosting eliminates or reduces these costs.
2. **Customization**: Configure the environment, tools, and resources as per project needs.
3. **Performance Control**: Monitor and manage resources directly.
4. **Always-On Availability**: Raspberry Pi can be set up to run 24/7, ensuring consistent availability for workflows.

### **Advantages of Raspberry Pi**

- **Low Power Consumption**: Ideal for running 24/7 with minimal electricity costs.
- **Compact Form Factor**: Fits easily in any workspace.
- **Affordable**: Raspberry Pi models (like the 4B) offer excellent value for self-hosting.
- **Open-Source Friendly**: Raspberry Pi supports various Linux-based OSes, ensuring compatibility with GitHub Actions.

---

## **Getting Started: Prerequisites**

### **Hardware Requirements**

- A Raspberry Pi 4B or newer (4GB RAM recommended for better performance).
- MicroSD card (16GB or larger) with a suitable OS.
- Power supply and cooling solution (optional but recommended for stability).

### **Software Requirements**

- **Raspberry Pi OS** (Lite version recommended for headless setup).
- GitHub Personal Access Token (PAT) with appropriate repository permissions.
- Docker (for containerized runners).
- GitHub Actions runner binary (provided by GitHub).

---

## **Step-by-Step Guide to Setting Up Raspberry Pi for GitHub Action Runners**

### **1. Prepare Your Raspberry Pi**

1. **Install Raspberry Pi OS**:
   - Download the latest [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/).
   - Use tools like [Raspberry Pi Imager](https://www.raspberrypi.org/software/) to flash the OS onto your microSD card.
2. **Enable SSH**:

   - Create an empty file named `ssh` in the `/boot` partition of the microSD card to enable SSH on boot.

3. **Boot the Raspberry Pi**:

   - Insert the microSD card into the Raspberry Pi, power it up, and connect it to your local network.

4. **Update and Upgrade Packages**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

---

### **2. Install Necessary Software**

1. **Install Docker**:
   Docker helps isolate the runner in a container, ensuring stability.

   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sh get-docker.sh
   sudo usermod -aG docker $USER
   ```

2. **Install Dependencies**:
   Install basic utilities required for the GitHub Actions runner.
   ```bash
   sudo apt install -y curl git
   ```

---

### **3. Configure the GitHub Actions Runner**

1. **Generate a GitHub PAT**:

   - Navigate to your GitHub account settings > Developer Settings > Personal Access Tokens.
   - Create a new PAT with `repo` and `workflow` scopes. Save this token securely.

2. **Download the Runner Software**:

   - Navigate to the repository or organization where you want the runner.
   - Go to Settings > Actions > Runners > New Self-Hosted Runner.
   - Copy the URL for the runner binary and download it onto the Raspberry Pi:
     ```bash
     mkdir actions-runner && cd actions-runner
     curl -o actions-runner-linux-arm64-2.308.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.308.0/actions-runner-linux-arm64-2.308.0.tar.gz
     tar xzf actions-runner-linux-arm64-2.308.0.tar.gz
     ```

3. **Configure the Runner**:

   ```bash
   ./config.sh --url https://github.com/your-repo-name --token YOUR_GITHUB_PAT
   ```

4. **Start the Runner**:
   ```bash
   ./run.sh
   ```

---

### **4. Set Up the Runner as a System Service**

To ensure the runner starts automatically after reboots:

1. **Create a Service File**:

   ```bash
   sudo nano /etc/systemd/system/github-runner.service
   ```

   Add the following content:

   ```ini
   [Unit]
   Description=GitHub Actions Runner
   After=network.target

   [Service]
   ExecStart=/home/pi/actions-runner/run.sh
   WorkingDirectory=/home/pi/actions-runner
   Restart=always
   User=pi

   [Install]
   WantedBy=multi-user.target
   ```

2. **Enable and Start the Service**:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable github-runner.service
   sudo systemctl start github-runner.service
   ```

---

### **5. Verify the Runner Setup**

1. Navigate to **Settings > Actions > Runners** in your GitHub repository.
2. Confirm that the runner is listed as **Online**.
3. Test the setup by triggering a GitHub Actions workflow in your repository.

---

## **Optimizing Performance on Raspberry Pi**

### **1. Use Swap Space**

Raspberry Pi’s limited RAM can be augmented with a swap file:

```bash
sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile
```

Increase `CONF_SWAPSIZE` to 2048 (2GB).

Restart the swap service:

```bash
sudo dphys-swapfile setup
sudo dphys-swapfile swapon
```

### **2. Use Cooling Solutions**

Install a fan or heatsink to prevent thermal throttling during high workloads.

### **3. Limit Concurrent Jobs**

Restrict the number of concurrent jobs to avoid overloading the Raspberry Pi:

```yaml
jobs:
  build:
    runs-on: self-hosted
    concurrency: 1
```

---

## **Diagrams and Supporting Images**

### **1. Network Diagram**

```plaintext
[GitHub Repo] -> [Raspberry Pi GitHub Runner] -> [Docker Container] -> [Action Execution]
```

### **2. Workflow Trigger Example**

Create a sample GitHub Actions YAML file:

```yaml
name: Build on Self-Hosted Runner

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: self-hosted
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      - name: Run Tests
        run: echo "Running Tests"
```

---

## **Conclusion**

Using a Raspberry Pi to self-host GitHub Action runners is a practical solution for developers seeking control, customization, and cost-efficiency in their CI/CD pipelines. With careful setup and optimization, you can leverage the Raspberry Pi’s versatility to handle workflows reliably. This guide provides all the steps necessary to transform your Raspberry Pi into a dependable GitHub Actions runner.

---

**FAQ**

### **1. Can I use older Raspberry Pi models?**

Yes, but models like the Raspberry Pi 3B+ may struggle with performance. Raspberry Pi 4B or newer is recommended.

### **2. Is Docker necessary for self-hosting?**

No, but it simplifies dependency management and sandboxing.

### **3. How many runners can I host on one Raspberry Pi?**

It depends on the workload and available resources. For heavy tasks, limit to one runner.

### **4. What happens if the runner goes offline?**

If you’ve set up the runner as a system service, it will automatically restart.

### **5. Can I self-host runners for multiple repositories?**

Yes, you can configure the runner for different repositories or organizations.

### **6. Is self-hosting secure?**

Yes, but ensure your Raspberry Pi is updated and protected behind a firewall.
