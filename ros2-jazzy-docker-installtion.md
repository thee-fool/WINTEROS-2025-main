
# Docker Installation and Running ROS Containers 

This guide provides step-by-step instructions to install Docker and run a ROS container on different operating systems: **Windows**, **macOS**, and **Ubuntu**.  

*Final Usage (once the whole setup is done) for Docker:* 
![12secfinal (1)](https://github.com/user-attachments/assets/0893ef22-5425-4e2c-a8d7-759f8319c895)


---
## Index
- [Windows Installation](#for-windows)
- [macOS Installation](#for-macos)
- [Ubuntu Installation](#for-ubuntu)
  
---


## For Windows  

### Step 1: Install WSL (Windows Subsystem for Linux)  

1. Open **PowerShell** as Administrator.  
2. Run the following command:  
   ```bash
   wsl --install
   ```
3. Once the installation is complete, reboot your system.  

> **Note**: If you already have WSL installed, you can skip this step and move to the next one.You can check if WSL is installed in your system by running `wsl --list --verbose` in the powershell. It will show the list of distros installed in WSL , if you have it already or else it will tell command not found. 

### Step 2: Install Docker Desktop  

1. Visit the [Docker installation page for Windows](https://docs.docker.com/desktop/setup/install/windows-install/).  
2. Download the `.exe` file for Docker Desktop.  
3. Run the installer and follow the on-screen instructions to complete the installation.  
4. After installation, reboot your system.  

### Step 3: Running the ROS Container  

1. Open **Docker Desktop** after logging into your system.  
2. In the bottom-right corner, open the **Docker Terminal**.
     
<img width="960" height="574" alt="1" src="https://github.com/user-attachments/assets/303908cb-6d15-41fa-97cf-e5eab3757f2e" />

   
4. Run the following command to start the ROS Docker container:  
   ```bash
   docker run -p 6080:80 --security-opt seccomp=unconfined --shm-size=512m adtyp/winteros_docker:jazzy
   ```
5. Wait for the container to initialize. This may take around 30 minutes for first-time users.  

> **Note**: The first run may take some time to download necessary files and set up the environment.

<img width="960" height="564" alt="2" src="https://github.com/user-attachments/assets/6fcfdc7d-d531-46fc-9d55-c6166867d8aa" />


### Step 4: Access ROS in the Browser  

1. Open your web browser.  
2. In the address bar, type `localhost:6080` and press Enter.  
3. You should now see your ROS environment running inside Docker.
4. To stop it , run `contol + C` in the terminal and to run the container again , just toggle the play button in the docker desktop , no need to folow the command line code again :))
   

<img width="960" height="564" alt="3" src="https://github.com/user-attachments/assets/1b882d23-e12b-407e-9f7f-970df12029c5" />


> **Note**: you have to only do this once (pasting the commad in the terminal) from the next time just toggle the play button in the containers section of docker desktop has shown in the above pic 


---

## For macOS  

### Step 1: Install XQuartz  

1. Download and install XQuartz from [here](https://www.xquartz.org/releases/XQuartz-2.8.1.html).  
2. Follow the on-screen instructions to complete the installation.  

### Step 2: Install Docker Desktop  

1. Visit the [Docker installation page for macOS](https://docs.docker.com/desktop/setup/install/mac-install/).  
2. Download the `.dmg` file for Docker Desktop.  
3. Run the `.dmg` file and follow the on-screen instructions.  

### Step 3: Running the ROS Container  

1. Open **Docker Desktop** after logging into your system.  
2. In the bottom-right corner, open the **Docker Terminal**.  
<img width="1372" alt="SCR-20241129-hdwg" src="https://github.com/user-attachments/assets/5f6891df-5274-4f3f-8b52-e6afcd59663d">

3. Run the following command to start the ROS Docker container:  
   ```bash
      docker run -p 6080:80 --security-opt seccomp=unconfined --shm-size=512m adtyp/winteros_docker:jazzy
   ```
4. Wait for the container to initialize. This may take around 30 minutes for first-time users.  

> **Note**: The first run may take some time to download necessary files and set up the environment.

<img width="1372" alt="image" src="https://github.com/user-attachments/assets/0cac3b54-d77f-4c06-b1e7-a021ba9349b6">



### Step 4: Access ROS in the Browser  

1. Open your web browser.  
2. In the address bar, type `localhost:6080` and press Enter.  
3. You should now see your ROS environment running inside Docker.
4. To stop it , run `contol + C` in the terminal and to run the container again , just toggle the play button in the docker desktop , no need to folow the command line code again :))
   <img width="1372" alt="image" src="https://github.com/user-attachments/assets/e68e95d1-2dc6-4fc3-807c-c8f91ab67086">

> **Note**: you have to only do this once (pasting the commad in the terminal) from the next time just toggle the play button in the containers section of docker desktop has shown in the above pic 
---

## For Ubuntu  

> **Note**: If you are using an environment other than GNOME, install GNOME Terminal using the following command:  
> ```bash
> sudo apt install gnome-terminal
> ```


### Step 1: Download Docker Desktop  

1. Download the `.deb` file for Ubuntu from the [Docker installation page](https://docs.docker.com/desktop/setup/install/linux/ubuntu/).  

### Step 2: Install Docker Desktop  

1. Open the terminal and enter the following commands:  
   ```bash
   sudo apt-get update
   sudo apt-get install ./docker-desktop-amd64.deb
   ```
2. Reboot your system.  

### Step 3: Running the ROS Container  

1. Open the **Terminal**.  
2. Run the following command to start the ROS Docker container:  
   ```bash
      docker run -p 6080:80 --security-opt seccomp=unconfined --shm-size=512m adtyp/winteros_docker:jazzy
   ```
3. Wait for the container to initialize. This may take around 30 minutes for first-time users.  

> **Note**: The first run may take some time to download necessary files and set up the environment.

<img width="1372" alt="image" src="https://github.com/user-attachments/assets/0cac3b54-d77f-4c06-b1e7-a021ba9349b6">


### Step 4: Access ROS in the Browser  

1. Open your web browser.  
2. In the address bar, type `localhost:6080` and press Enter.  
3. You should now see your ROS environment running inside Docker.
4. To stop it , run `contol + C` in the terminal and to run the container again , just toggle the play button in the docker desktop , no need to folow the command line code again :))
   <img width="1372" alt="image" src="https://github.com/user-attachments/assets/e68e95d1-2dc6-4fc3-807c-c8f91ab67086">

> **Note**: you have to only do this once (pasting the commad in the terminal) from the next time just toggle the play button in the containers section of docker desktop has shown in the above pic 
---
## ⚡ ALL SET
<p align="center">
  <img src="https://github.com/user-attachments/assets/9be17ef2-fbf3-4b5d-b540-536e87a4cf18" width="450">
</p>





