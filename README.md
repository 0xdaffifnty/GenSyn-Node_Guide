**Gensyn RL Swarm Node Setup Guide (for Anyone Using Google Cloud + Free Credits)**

_Welcome! This guide will walk you through installing Gensyn RL Swarm node on Google Cloud (GCP) VM using free $300 credits!_


_**1. Create a Google Cloud Account & Claim Free Credit**_

    Visit: https://cloud.google.com
    Sign up, verify your identity (debit/credit card required)
    You’ll receive $300 free credits valid for 90 days



_**2. Create a VM on Google Cloud (to run the Gensyn node)**_

   From GCP Console:

    Go to Compute Engine → VM instances → Create Instance

    Configure:

    Name: gensyn-node

    Region: Your nearest

    Machine type: n2-standard-8 (8 vCPU, 32 GB RAM) 

    Boot disk: Ubuntu 22.04 LTS, 50GB SSD

    Firewall: ✅ Check "Allow HTTP" and "Allow HTTPS"

    Under Security → SSH Keys, paste the output of the SSH public key (next step)

   Click Create

   

**3. _Allow Gensyn Dashboard Port (3000) Access on GCP----> The Gensyn node runs a web dashboard on port 3000. By default, Google Cloud blocks all incoming ports except HTTP/HTTPS (80/443). So you need to manually allow traffic on port 3000 using a Firewall Rule._**

Here's how to open port 3000:

    Step 1: Go to VPC Network

    In your Google Cloud Console sidebar (left menu), go to: VPC Network > Firewall

    Step 2: Create a New Firewall Rule

    Click the blue “CREATE FIREWALL RULE” button at the top

    Step 3. Use the following settings:

    Name--> aloow-gensyn-3000

    Network--> Default

    priority--> 1000 (leave default)

    Direction of traffic--> Ingress

    Action on Match--> Allow

    Targets--> All instances in the network

    Source Filter--> IPv4 ranges

    Source IPv4 ranges--> 0.0.0.0/0 (allows access from anywhere)

    Protocols & ports--> Check “Specified protocols and ports

    Then check TCP and type: 3000

    Step 4: Click Create

Wait a few seconds — the rule will be applied globally to any VM on the default network (your VM is already there by default unless you changed it).



_**4. Create Your Own SSH Key on Your Local PC**_

   Open your terminal (Command Prompt, WSL, Git Bash, or Terminal on Mac/Linux) & run this command:

    ssh-keygen -t rsa -b 4096 -C "your_email@gmail.com"

   When it asks:

    Enter file in which to save the key: Just press Enter to use default (~/.ssh/id_rsa)

    Set a passphrase (optional, you can skip by pressing Enter)

   This creates:

    Private key → ~/.ssh/id_rsa

    Public key → ~/.ssh/id_rsa.pub

** You should put the email you used to signup with google cloud

_**Then follow this to view your public key:**_

   Option 1: Use PowerShell to print it

   Open PowerShell (not Command Prompt)

   Run:

    Get-Content $env:USERPROFILE\.ssh\id_rsa.pub

    This will print the entire public key — just copy all of it and paste it into your GCP VM

   Option 2: Open the file manually

   Go to:

    C:\Users\YourUsername\.ssh\

    Right-click on id_rsa.pub and open with Notepad

    Copy the full line — starts with ssh-rsa and ends with your email

   Once copied, go back to Google Cloud → your VM → Edit → SSH keys → paste it as a new item and remove the old ones.



_**5. Connect to Gensyn VM via SSH**_

   Open Powershell & run this:

    ssh -i C:\Users\"YOUR-USERNAME"\.ssh\id_rsa your-username@your-external-ip

  -i ~/.ssh/id_rsa → This tells SSH to use your private key

  your-username → The Linux username linked to your SSH key on the VM

  your-external-ip → The External IP address of your Google Cloud VM

  
    **Note** _"C:\Users\"YOUR-USERNAME"---> You need to put the username of your PC here_**
  
    **Note** _If you stop/restart the VM on GCP the External IP will be changed, to reserve it go to GCP---> VPC Network----> IP Addresses---> External IP---> Click & Reserve External IP_**




_**6. Install the Gensyn RL Swarm Node (inside the VM)**_

Step 1: Update the system

    sudo apt update && sudo apt upgrade -y

Step 2: Install dependencies

    sudo apt install -y \

    curl git wget nano tmux htop nvme-cli lz4 jq make gcc clang \

    build-essential autoconf automake pkg-config libssl-dev libleveldb-dev \

    libgbm1 bsdmainutils ncdu unzip tar

Step 3: Install Python

    sudo apt install python3 python3-pip -y
    sudo apt install python3-venv python3-dev build-essential -y

Step 4: Install Node.js & Yarn

    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

    sudo apt install -y nodejs

    sudo npm install -g yarn

Step 5: Install Docker

    for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do

    sudo apt-get remove $pkg
  
    done

      sudo apt-get update

      sudo apt-get install -y ca-certificates curl gnupg

      sudo install -m 0755 -d /etc/apt/keyrings

      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

      sudo chmod a+r /etc/apt/keyrings/docker.gpg

    echo \

      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  
      https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

      sudo apt-get update

      sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

 Step 6: Install screen

    sudo apt install screen -y

**_6. Clone and Run the Gensyn Node_**

Inside the VM:

    git clone https://github.com/gensyn-ai/rl-swarm

    cd rl-swarm

    python3 -m venv .venv

    source .venv/bin/activate

    screen -S rlswarm

    ./run_rl_swarm.sh

It will ask you Do you want to connect to testnet [Yes/No]

    ----> Press Y

Connect Huggingface API [Yes/No]

    ----> Press Y

    **Note** _Create a huggingface access token---> Visit huggingface.co----> under your profile drop down look for access token----> create a gensyn access token with "Write" permission_**


Detach screen: Press Ctrl + A, then D

Resume screen: screen -r rlswarm

Stop node: screen -r rlswarm, then press Ctrl + C


    **Note** _When you see "USER JSON DATA TO BE CREATED" output that means you need to sign in to Gensyn dashboard from your PC browser_**



**7. _Access Gensyn Dashboard from Your Local Browser_**

From your local PC:

Open another terminal tab (Powershell/CMD) and run:

    ssh -i C:\Users\"YOUR_USERNAME"\.ssh\id_rsa -L 3000:localhost:3000 YOUR_USERNAME@VM_EXTERNAL_IP

This command forwards port 3000 from your VM to your PC's localhost:3000

Now open this URL in your local browser:

    http://localhost:3000

You can now:

Enter your email

Receive OTP

Log in to the Gensyn dashboard



8. Backup SWARM.pem File (Very Important)

Step 1: SSH Into the Gensyn VM from Your PC-->

If you're on Windows using CMD or PowerShell and it's saved in the default Windows path, use:

    ssh -i C:\Users\"YOUR-USERNAME"\.ssh\id_rsa your-username@your-external-ip

Step 2: Locate the swarm.pem File

Once inside the Gensyn VM run:

    find / -name swarm.pem 2>/dev/null

This command searches your entire filesystem (ignore permission errors)

    Usually, it’s located somewhere like:

    /home/"USER"/.gensyn/swarm.pem

    Note** User name is the GCP SSH username (You can look for the username under GCP VM, scroll down & look for the SSH keys & username) 

Step 3: Securely Copy swarm.pem to Your Local Computer

From your local terminal, exit the VM (Ctrl + D or exit) and run:

    scp -i C:\Users\"YourUsername"\.ssh\id_rsa YOUR_USERNAME@VM_EXTERNAL_IP:/home/"user"/.gensyn/swarm.pem C:\Users\YourUsername\Downloads\

**_Note** if your computer user name is: "mydesktop" & GCP gensyn VM username is "GCPGENSYN" & External Ip is xx.xx.xx.xx then the command would look like this:

    scp -i C:\Users\mydesktop\.ssh\id_rsa GCPGENSYN@xx.xx.xx.xx:/home/"GCPGENSYN"/.gensyn/swarm.pem C:\Users\YourUsername\Downloads\ (it's an example)
    
This will copy the swarm.pem file to your local Downloads folder.

(You can change ~/Downloads/ to wherever you want to save it.)



**_9. Ensure You Use Only Free GCP Credit & Avoid Charges_**

Setup Billing Alerts:

    Go to Billing → Budgets & Alerts

    Create a new budget for $300

    Enable email alerts (e.g. at $200, $250)

    Avoid charges after you're done:

    Go to Compute Engine → VM instances

    Click “Stop” next to your VM

You won’t be charged for CPU/RAM while the VM is off. (Optional: Delete disks and project when done, if you want to be absoultely sure)

**NOTE****  **_IF YOU PLAN TO RUN THE NODE ONLY ON THE FREE CREDITS & DO NOT WANT TO BE CHARGED THEN YOU MUST SET A BUDGET ALERT AND BEFORE USING ALL THE FREE CREDITS YOU SHOULD STOP THE NODE & DELETE YOUR PAYMENT METHOD & BILLING INFO FROM GOOGLE CLOUD CONSOLE_**

