# 🚀 Network Auto-Backup Setup Guide (Cisco, MikroTik, Huawei, Juniper) with n8n & Telegram

🌍 **Read this in other languages:** [English](README.md) | [Bahasa Indonesia](README_id.md)

Welcome! This workflow will turn your Telegram into a personal network assistant. You can add routers/switches, remove them, and have the system perform automatic or manual backups—all right through Telegram chat!

This system uses n8n (an automation platform) and supports 4 of the most popular network device brands: MikroTik, Cisco, Huawei, and Juniper.

Let's get started with the setup steps! No coding skills required, just follow the easy guide below.

---

## 📋 Initial Preparation (What You Need)

Before starting, make sure you have prepared these 3 things:
1. **n8n Application:** Installed and running (can be on Docker, Cloud, or Desktop).
2. **Telegram Bot:** Create a new bot via [@BotFather](https://t.me/BotFather) on Telegram and save its API Token.
3. **FTP Server:** A place (server/PC) running an FTP service to store backup files from routers.

---

## 🛠️ Step 1: Create a Database (Data Table)

The system needs a place to record the list of routers you own. We will use n8n's built-in database feature.

1. Open your n8n. In the left menu, click **Data tables**.
2. Click the **New data table** button. Name it anything you like (e.g., `data auto backup`).
3. You can import the sample (dummy) file I provided, or create the columns manually. If manually, create the following columns (pay attention to lowercase letters):
   * `address` (Type: String)
   * `port` (Type: Number)
   * `username` (Type: String)
   * `password` (Type: String)
   * `brand` (Type: String)
   * `portftp` (Type: Number)
   * `date_last_backup` (Type: Date & Time)

<img width="837" height="73" alt="image" src="https://github.com/user-attachments/assets/12264ad5-b2fd-4a5f-a53a-524a0b8152b5" />

---

## 📥 Step 2: Import Workflow to n8n

Now let's insert the automation "brain" into n8n.

1. In the left n8n menu, click **Workflows**, then click **Add workflow**.
2. In the top right corner, click the options button (three dots `...`), then select **Import from File**.
3. Choose the workflow `.json` file that you downloaded.

Tadaa! The automation network will appear on your screen.

<img width="626" height="735" alt="image" src="https://github.com/user-attachments/assets/88eb96d0-5a05-4ecf-bac4-5349d282278f" />

---

## 🔗 Step 3: Connect Accounts (Credentials) & Table

Since this is a new workflow, you need to "introduce" your Telegram account and FTP server to n8n, and connect the table created in Step 1.

### A. Connecting the Table
1. Find all nodes named **Data Table** (orange table icon).
2. Double-click (open) those nodes one by one.
3. Under the **Data table** section, click the dropdown menu and select the table name you created in Step 1 (e.g., `data auto backup`).
4. Repeat for all Data Table nodes so the system knows where to read and store data.

### B. Entering the Telegram Token
1. Find the node named **Telegram Trigger** (on the far left) and double-click it.
2. In the **Credential** section, click the downward arrow and select **Create New Credential**.
3. Enter the API Token you obtained from `@BotFather`. Save it.
4. Repeat this Telegram credential selection for all nodes featuring the Telegram logo (blue color).

### C. Filling in External FTP Credentials
1. Find the FTP node named **Upload Backup to Server** (or **Download Backup from Server**).
2. Create a new FTP credential. Enter the IP Address, Username, and Password of the FTP server where you want to store the final backup files.

<img width="807" height="375" alt="image" src="https://github.com/user-attachments/assets/50cbb6dd-6cd5-4d6c-acf5-35e5b7763e0c" />

---

## ⚙️ Dynamic SSH and FTP Credentials Setup

The beauty of this workflow is that you don't need to manually create SSH and FTP credentials one by one for each device. The system is designed using the Dynamic Credentials feature in n8n, where access information (Host, Port, Username, and Password) is pulled automatically straight from the row of data being processed in the database table. 

When configuring SSH and FTP credentials in n8n, switch the input fields to Expression mode (`fx`) and enter the following variables:

* **For SSH Nodes:**
  * Host: `{{ $json.address }}`
  * Port: `{{ $json.port }}`
  * Username: `{{ $json.username }}`
  * Password: `{{ $json.password }}`

* **For FTP Nodes:**
  * Host: `{{ $('Get Credential').item.json.address }}`
  * Port: `{{ $('Get Credential').item.json.portftp }}`
  * Username: `{{ $('Get Credential').item.json.username }}`
  * Password: `{{ $('Get Credential').item.json.username }}`

<img width="423" height="460" alt="image" src="https://github.com/user-attachments/assets/5169e1e0-320b-48a2-9ba0-523cb43095e0" />
<img width="583" height="573" alt="image" src="https://github.com/user-attachments/assets/873f83fe-e6e8-4d7b-8b38-27fe73eac969" />
<img width="539" height="570" alt="image" src="https://github.com/user-attachments/assets/f02bb63b-d578-4158-a2a3-4753864cbc80" />

> ⚠️ *Ignore any errors shown in the Expression result for this step, as that is expected behavior; however, it will function 100% when the node is active.*

---

## 🚀 Step 4: Activate the Workflow!

Finished setup? It's time to turn on the engine!

1. In the top right corner of the n8n canvas, toggle the **Publish** button (from gray to green/ON).
2. n8n is now on standby 24/7 to listen for commands from your Telegram.

<img width="702" height="135" alt="image" src="https://github.com/user-attachments/assets/7b453e9f-1a27-41dd-9ab2-d748ac6d1fcd" />

---

## 📱 How to Use in Telegram

Now open the Telegram application and open a chat with your Bot. Type these magic commands:

### 1. Adding a New Device (`/add`)
* Send a message to the bot with the following format (separated by spaces):
  `/add [IP_Address] [Port_SSH] [Username] [Password] [Brand] [Port_FTP]`
* **Example:**
  `/add 192.168.1.1 22 admin secret123 mikrotik 21`
* The bot will immediately reply that the device was successfully added and present a list of active devices. *(Note: Supported brands are only mikrotik, cisco, huawei, and juniper).*

<img width="705" height="246" alt="image" src="https://github.com/user-attachments/assets/c9f7c22f-8a55-4a8b-9ae8-51fa9c09865f" />

### 2. Viewing the Main Menu (`/menu`)
* Type `/menu`. The bot will display a greeting text along with two interactive buttons:
  * **Backup Sekarang**: Forces the system to perform a backup for all devices in the list right away!
  * **Download Backup Terakhir**: Requests the latest backup file to be sent directly to your Telegram.

<img width="704" height="230" alt="image" src="https://github.com/user-attachments/assets/b1ede717-da52-46be-bd79-66fdf90e6d22" />
 
### 3. Removing a Device (`/remove`)
* Want to see the device list along with their ID numbers? Just type `/remove`.
* Want to delete device number 2? Type `/remove 2`. The bot will instantly remove it from the database.

<img width="713" height="612" alt="image" src="https://github.com/user-attachments/assets/ca0f6e4d-7bef-4c0e-8705-e3e607229ebc" />

---

## 💡 Additional Notes (Important!)

1. **Automatic Schedule:** The system automatically performs backups on the 1st of every month at 02:00 AM. You can change this schedule by editing the **Schedule Trigger** node (clock logo at the top).
2. **For Cisco & Huawei:** Make sure the username you use for SSH has the highest level of access rights (Privilege Level 15 / Super Admin) so the n8n system can execute backup commands directly without being blocked by additional password prompts.
3. **FTP Feature on Routers:** Make sure the FTP Server feature inside each router/switch you register is already active (Enabled), because n8n will retrieve backup files from the device via the FTP path.

Good luck! If an error occurs, the Telegram bot will automatically send an error notification along with the name of the problematic step for easier troubleshooting.
