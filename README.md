# Lab 1: Running Algonquin Pet Store on an Azure VM

Welcome to the **Algonquin Pet Store** application! This app will help you learn about how modern web applications work by combining several technologies into a working system. The application is built using microservices architecture, which means that different parts of the app work independently but communicate with each other. So, instead of one giant application, we break things down into smaller, independent services that work together.
Think of it like a team:

- One service manages the store’s website
- Another handles orders
- Another keeps track of products
- And a special tool makes sure they can all talk to each other smoothly

In this lab, you will learn how to run the Algonquin Pet Store on an Azure Virtual Machine and locally (Optional). This hands-on exercise will teach you how to run a multi-service application and work with cloud VM.

![App UI](./Docs/app-ui.png)

## Lab Objectives:
- Set up and run the application on an Azure VM.
- Set up and run the application on your local machine (Optional).

## Important Note:
Before you start the lab, **please read through the entire instructions carefully.** It is important to understand the flow and requirements fully before beginning the tasks. If you are working in a group, make sure all team members are familiar with the instructions and assigned responsibilities.

## Group Work Recommendation
- You are encouraged to work in pairs (groups of 2)
- However, each student must prepare and submit their own work

## Prerequisites:
- **Azure Account**: Make sure you have access to a Microsoft Azure account.
- **VS Code**: Install [Visual Studio Code](https://code.visualstudio.com/).
- **WSL (Windows Users)**: If you're using Windows, it is recommended to use **Windows Subsystem for Linux (WSL)** for your local development environment.
---

## Architecture Overview
Here’s the big picture:
![App Architecture](./Docs/app-architecture.png)

The app uses a microservices architecture. This means that different parts of the app (called "services") do specific jobs and talk to each other to get things done. Here's how they fit together:

- **Store Front (Vue.js)**: A front-end application where customers can browse and order products.
- **Order Service (Node.js)**: Manages customer orders and interacts with RabbitMQ for message queuing.
- **Product Service (Rust)**: Handles product listings. Manages product details and inventory.
- **RabbitMQ**: Message broker for communication between services. Used to queue orders for processing.

## Instructions
### Step 1: Setting Up an Azure Virtual Machine
- Go to the Azure portal.
- Click Create a Resource and select Virtual Machine.
- Choose the following configurations:
  - Image: Choose Ubuntu 24.04 or 22.04 LTS.
  - Size: Select an appropriate VM size (e.g., Standard B2s) for small-scale testing.
  - Authentication: Use an SSH public key for secure access.
    - When you creat your Azure VM, you will download a private key file (usually named something like my-azure-key.pem).
    - Keep this file in a safe place on your computer, for example in a folder like `~/.ssh/` (on Mac/Linux) or `C:\Users\<YourName>\.ssh\` (on Windows).  

Once your VM is created, note down the public IP address for future use.

### Step 2: Configure the Network Security Group (NSG)
- When you create a Virtual Machine in Azure, by default it is **locked down for security**. That means people (including you) cannot access the services running on the VM unless you explicitly allow them.  
- This is where the **Network Security Group (NSG)** comes in.  
Think of an NSG like a **firewall**: it controls which traffic is allowed **in** (Inbound Rules) and which traffic is allowed **out** (Outbound Rules).  
- Since our application runs multiple services on different ports, we must tell Azure to open those ports so we can access them from our browser.  
- After creating the VM, you need to configure the **Network Security Group (NSG)** attached to the VM to allow access to specific ports.


- Go to the **Networking** section of your VM in the Azure portal.
- Make sure the following ports are open in your VM’s Network Security Group by creating the following **Inbound Port Rules**:
  - 8080 → Store Front
    - This is the website (Vue.js front-end). If this port isn’t open, you won’t see the pet store in your browser.  
  - 3000 → Order Service
    - The back-end service that processes orders. The front-end calls this API, so it must be accessible.  
  - 3030 → Product Service
    - The service that provides product details and inventory. The front-end talk to it, so it must be open.  
  - 15672 → RabbitMQ management dashboard (optional, for admin use)  
    - A web-based interface where you (as admin) can see message queues, exchanges, and monitor RabbitMQ. Students may use this to debug and learn, but it’s not required for customers using the store.  

### Step 3: Setting Up VS Code for Remote Development on Azure VM
Instead of using a regular SSH terminal, you can use VS Code to remotely access the VM, making it easier to work on files and manage services.

- In VS Code, open the Extensions view (Ctrl + Shift + X). Search for and install the `Remote - SSH extension`.
- Open the Command Palette (Ctrl + Shift + P)
- Search for `Remote-SSH: Add New SSH Host`
- Enter the SSH command in this format:
  ```
  ssh -i /path/to/my-azure-key.pem azureuser@<Public-IP>
  ``` 
  - Replace `/path/to/my-azure-key.pem` with the full path to your downloaded .pem file.
  - Replace `azureuser` with the username you set when creating the VM.
  - Replace `<Public-IP>` with your VM’s public IP address.
  - Example (Windows): `ssh -i C:\Users\YourName\.ssh\my-azure-key.pem azureuser@20.50.100.200`
  - VS Code will ask where to save this configuration, usually `C:\Users\YourName\.ssh\config`. This makes it easier to connect in the future.
- Open the Command Palette again and select `Remote-SSH: Connect to Host…`.
- Choose the host you just added.
- VS Code will open a new window connected directly to your Azure VM.


### Step 4: Install Dependencies on the VM
Once connected to the Azure VM through VS Code:
- Open a new terminal in VS Code (Ctrl + Shift + P, then type Terminal: Create New Terminal).
- Clone the Repository on the VM
- Before you can run the full app, you’ll need to set up each part. Don’t worry if these technologies are new, here’s a quick overview of what each one does and why we’re using it:

#### RabbitMQ
- A **message broker** (like a post office). Services drop messages into a queue, and RabbitMQ delivers them in the right order.  
- Makes sure messages (like “Customer placed an order of 10 dog food”) don’t get lost.  
- Follow the instructions in the [RabbitMQ README](RabbitMQ/README.md) to get RabbitMQ running.

#### Order-Service
- Handles **customer orders** (e.g., “10 bags of dog food”).  
- Written in **Node.js**, which lets us build server-side apps in JavaScript. 
- Navigate to the `order-service` folder and follow the instructions in its [Order-Service README](order-service/README.md).

#### Product-Service
- Keeps track of **products**: names, prices, and stock levels.  
- Written in **Rust**, a fast and safe programming language great for reliable, high-performance services.  
- Navigate to the `product-service` folder and follow the instructions in its [Product-Service README](product-service/README.md).

#### Store-Front
- The **website** customers use to shop.  
- Built in **Vue.js**, a beginner-friendly framework for interactive web pages.
- Navigate to the `store-front` folder and follow the instructions in its [Store-Front README](store-front/README.md).

## Step 5: Running the Full Application  

Once all the services are running, you can view the app in your browser.  

If your VM’s IP is `20.51.123.45`, open: [http://20.51.123.45:8080](http://20.51.123.45:8080)  

---

## Step 6: Running the app on your local machine (Optional) 
- Repeat step 4 on your local machine to install depedndecies. 
- Open your browser and go to:  
  [http://localhost:8080](http://localhost:8080)  
- The **Store Front** will load and show product listings from the **Product Service**.  
- When you place an order:  
  - The **Order Service** processes it  
  - **RabbitMQ** makes sure the message is delivered safely to other services. 


 That’s it! Whether you’re running locally or in the cloud, you now have a working microservices-based pet store app.

---

## Submission

For this first lab, you will demonstrate both your ability to get the system running and your understanding of the individual services.

### What to Submit
1. **Demo Video (Max 5 minutes)**
   - Record a short demo video showing your running application.  
   - At minimum, your demo should include:
     - The application running on your Azure VM (store-front accessible in the browser).
     - A product listing being displayed from the Product Service.
     - An order being placed through the Order Service.
   - Upload your video to **YouTube** (unlisted is fine).
   - Copy the video link, you will include this in your `README.md` file.

2. **Brief Technical Explanation**
   - Spend some time examining the source code of each service:
     - **Order Service (Node.js)**
     - **Product Service (Rust)**
     - **Store Front (Vue.js)**
   - For each service, write a **short technical explanation** (1–2 paragraphs each):
     - What the service is responsible for.
     - Which language/framework it uses.
     - How it interacts with the other services (e.g., RabbitMQ, APIs, or front-end).
   - Keep it simple but show that you understand the role of each service.

3. **GitHub Repository**
   - Create a **GitHub repository** for your submission.
   - Your repository must include:
     - A `README.md` file with:
       - The YouTube video link.
       - Your written technical explanations for the three services.
     - (Optional) Any notes you want to share about setup challenges or learnings.

### How to Submit
- Push your work to a public GitHub repository.
- Include the YouTube demo link and explanations in the `README.md`.
- Submit the link to your GitHub repository as your final lab deliverable in **Brightspace**.
