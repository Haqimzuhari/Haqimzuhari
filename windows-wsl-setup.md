# 🚀 Windows WSL

## 📌 Overview
This guide provides a step-by-step approach to setting up **Docker Desktop** and **Windows Subsystem for Linux (WSL)** on Windows. It also includes instructions for creating a WSL instance (Ubuntu 20.04) and connecting to it using **VS Code**.

---

## 🛠️ Step 1: Install Docker Desktop

1. Download **Docker Desktop** from the [official website](https://www.docker.com/products/docker-desktop/).
2. Run the installer and follow the on-screen instructions.
3. Ensure that **WSL integration** is enabled during installation.
4. After installation, restart your computer.
5. Open **Docker Desktop** and verify it's running correctly.

To check if Docker is installed, run:
```sh
docker --version
```

---

## 🏗️ Step 2: Install Windows Subsystem for Linux (WSL)

1. Open **PowerShell** as Administrator.
2. Run the following command to enable WSL:
```powershell
wsl --install
```
3. Restart your computer.
4. Verify installation by running (which also to view installed WSL):
```powershell
wsl --list --verbose
```
or
```powershell
wsl -l -v
```

---

## 🐧 Step 3: Create a WSL Instance (Ubuntu 20.04 Example)

Open **PowerShell** or **cmd** list available WSL:   
```powershell
wsl --list --online
```
or
```powershell
wsl -l -o
```
Result is as below
```powershell
The following is a list of valid distributions that can be installed.

NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Ubuntu-18.04                    Ubuntu 18.04 LTS
Ubuntu-20.04                    Ubuntu 20.04 LTS
...
```
Run below command to install WSL instance
```powershell
wsl --install [WSL-name] # Example: wsl --install Ubuntu-20.04
```
Usually for first time install, you will auto logged into WSL   
&nbsp;

In case you accidently logged out and want to login into existing WSL   
Follow below step:
```powershell
wsl -l -v  # To list all installed WSL
wsl -d [WSL-name] # Log into WSL
```

---

## 🔥 Step 4: First Things to Do After Logging into WSL

After logging into your WSL instance:

1. **Update the package list & upgrade installed packages:**
```sh
sudo apt update && sudo apt upgrade -y
```
2. **Install essential tools:**
```sh
sudo apt install -y curl git unzip build-essential
```
3. **Verify Docker integration with WSL:**
```sh
docker --version
```

---

## 💻 Step 5: Remote WSL Using VS Code

Install [Visual Studio Code](https://code.visualstudio.com/).
Install the **WSL** and **Remote - Explorer** extension from the VS Code marketplace.
Open VS Code and press `F1`, then select:
```
WSL: Connect to WSL using Distro..
```
Choose your WSL instance.
Open your project folder explorer.   

Now you can develop inside WSL seamlessly using VS Code! 🎉

---

## 🎯 Conclusion
By following these steps, you've successfully:
✅ Installed Docker Desktop 🐳  
✅ Set up WSL with Ubuntu 20.04 🐧  
✅ Configured Docker within WSL 🛠️  
✅ Connected to WSL using VS Code 💻  

You're now ready to build and deploy your projects using a powerful **WSL Docker development environment**! 🚀   
Start your first docker instance by following below guideline   
🔗 **Check out the docker-pnxm repository:** [docker-pnxm](https://github.com/Haqimzuhari/docker-pnxm)

