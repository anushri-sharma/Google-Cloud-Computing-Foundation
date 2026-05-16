GSP322

# [Build a Secure Google Cloud Network: Challenge Lab](https://www.skills.google/paths/36/course_templates/654/labs/592556)

[Link](https://www.youtube.com/watch?v=y4NQp3PPz5k&time_continue=926&source_ve_path=NzY3NTg&embeds_referring_euri=https%3A%2F%2Fwww.google.com%2Fsearch%3Fsca_esv%3D17e9b9252c193486%26udm%3D7%26sxsrf%3DANbL-n4EIK276wvwiRbzdunJnzCYHdQKuQ%3A1778765166181%26q%3DBuild%2Ba%2BSe&themeRefresh=1)

# Introduction
In a challenge lab you’re given a scenario and a set of tasks. Instead of following step-by-step instructions, you will use the skills learned from the labs in the course to figure out how to complete the tasks on your own! An automated scoring system (shown on this page) will provide feedback on whether you have completed your tasks correctly.

When you take a challenge lab, you will not be taught new Google Cloud concepts. You are expected to extend your learned skills, like changing default values and reading and researching error messages to fix your own mistakes.

To score 100% you must successfully complete all tasks within the time period!

This lab is recommended for students who have enrolled in the **Build a Secure Google Cloud Network skill badge**](https://www.skills.google/paths/36/course_templates/654/labs/592556). Are you ready for the challenge?

# Challenge scenario
You are a security consultant brought in by Jeff, who owns a small local company, to help him with his very successful website (juice-shop). Jeff is new to Google Cloud and had his neighbour's son set up the initial site. The neighbour's son has since had to leave for college, but before leaving, he made sure the site was running.

Below is the current set up:

<img width="630" height="408" alt="image" src="https://github.com/user-attachments/assets/d73e4ef8-b0d7-4d75-b1e9-c775b6717eb3" />

# Your challenge
You need to create the appropriate security configuration for Jeff's site. Your first challenge is to set up firewall rules and virtual machine tags. You also need to ensure that SSH is only available to the bastion via IAP.

## For the firewall rules, make sure that:

- The bastion host does not have a public IP address.
- You can only SSH to the bastion and only via IAP.
- You can only SSH to juice-shop via the bastion.
- Only HTTP is open to the world for juice-shop.

### Tips and tricks:

- Pay close attention to the network tags and the associated VPC firewall rules.
- Be specific and limit the size of the VPC firewall rule source ranges.
- Overly permissive permissions will not be marked correct.

<img width="640" height="415" alt="image" src="https://github.com/user-attachments/assets/c5cb37cb-2c86-435f-8421-4d30c28bd00a" />

## Suggested order of action.

> 1. Check the firewall rules. Remove the overly permissive rules.

> 2. Navigate to Compute Engine in the Cloud console and identify the bastion host. The instance should be stopped. Start the instance.
> 3. The bastion host is the one machine authorized to receive external SSH traffic. Create a firewall rule that allows [**SSH (tcp/22) from the IAP service**](https://docs.cloud.google.com/iap/docs/using-tcp-forwarding). The firewall rule must be enabled for the bastion host instance using a network tag of SSH IAP network tag.
> 4. The juice-shop server serves HTTP traffic. Create a firewall rule that allows traffic on HTTP (tcp/80) to any address. The firewall rule must be enabled for the juice-shop instance using a network tag of HTTP network tag.
> 5. You need to connect to juice-shop from the bastion using SSH. Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet network address. The firewall rule must be enabled for the juice-shop instance using a network tag of SSH internal network tag.
> 6. In the Compute Engine instances page, click the SSH button for the bastion host. Once connected, SSH to juice-shop.
>>>**Hint:** If you're having difficulties with the compute ssh connection or IAP tunnel, make use of the --troubleshoot flag.
>>>

## 1. Check and Remove Overly Permissive Firewall Rules

1. Open [Google Cloud Console](https://console.cloud.google.com?utm_source=chatgpt.com)
2. Go to:

   * Google Cloud → **VPC network** → **Firewall**
3. Look for overly permissive rules such as:

   * `0.0.0.0/0`
   * Allowing all ports/protocols
   * Wide SSH access
4. Select the insecure rules.
5. Click **Delete**.

---

# 2. Start the Bastion Host

1. Navigate to:

   * **Compute Engine** → **VM instances**
2. Find the VM named similar to:

   * `bastion`
3. Status should be:

   * **Terminated / Stopped**
4. Select the instance.
5. Click **Start**.
6. Wait until status changes to:

   * **Running**

---

# 3. Create Firewall Rule for IAP SSH Access to Bastion

This allows SSH only through Identity-Aware Proxy.

## Steps

1. Go to:

   * **VPC network** → **Firewall**
2. Click **Create Firewall Rule**

## Configure

### General

* **Name**:

```text
allow-ssh-iap
```

* **Network**:
  Select your lab VPC network.

* **Direction of traffic**:

```text
Ingress
```

* **Action on match**:

```text
Allow
```

* **Targets**:

```text
Specified target tags
```

* **Target tags**:

```text
ssh-iap
```

---

### Source Filter

* **Source IPv4 ranges**:

```text
35.235.240.0/20
```

(This is the official IAP TCP forwarding range.)

---

### Protocols and Ports

Select:

```text
Specified protocols and ports
```

Check:

```text
tcp
```

Port:

```text
22
```

3. Click **Create**.

---

# 4. Add Network Tag to Bastion Host

1. Go to:

   * **Compute Engine** → **VM instances**
2. Click the bastion VM.
3. Click **Edit**.
4. Under **Network tags**, add:

```text
ssh-iap
```

5. Click **Save**.

---

# 5. Create Firewall Rule for HTTP Access to juice-shop

## Steps

1. Go to:

   * **VPC network** → **Firewall**
2. Click **Create Firewall Rule**

## Configure

### General

* **Name**

```text
allow-http
```

* **Targets**

```text
Specified target tags
```

* **Target tags**

```text
http-server
```

---

### Source Filter

* **Source IPv4 ranges**

```text
0.0.0.0/0
```

---

### Protocols and Ports

Allow:

```text
tcp:80
```

3. Click **Create**.

---

# 6. Add HTTP Network Tag to juice-shop

1. Open:

   * **Compute Engine** → **VM instances**
2. Click the `juice-shop` VM.
3. Click **Edit**.
4. Under **Network tags**, add:

```text
http-server
```

5. Click **Save**.

---

# 7. Create Internal SSH Firewall Rule for juice-shop

This allows SSH only from the management subnet.

## Steps

1. Go to:

   * **VPC network** → **Firewall**
2. Click **Create Firewall Rule**

## Configure

### General

* **Name**

```text
allow-ssh-internal
```

* **Targets**

```text
Specified target tags
```

* **Target tags**

```text
ssh-internal
```

---

### Source Filter

* **Source IPv4 ranges**

Use the **acme-mgmt-subnet** CIDR range from:

* **VPC network** → **VPC networks** → subnet details

Example:

```text
10.0.1.0/24
```

(Use the exact subnet range shown in your lab.)

---

### Protocols and Ports

Allow:

```text
tcp:22
```

3. Click **Create**.

---

# 8. Add SSH Internal Tag to juice-shop

1. Open `juice-shop` VM.
2. Click **Edit**.
3. Under **Network tags**, add:

```text
ssh-internal
```

4. Click **Save**.

---

# 9. SSH into Bastion Host

1. Go to:

   * **Compute Engine** → **VM instances**
2. Click **SSH** beside the bastion VM.

A terminal window opens.

---

# 10. SSH from Bastion to juice-shop

Inside the bastion terminal:

## Find internal IP of juice-shop

Run:

```bash
gcloud compute instances list
```

Copy the **INTERNAL_IP** of `juice-shop`.

---

## SSH into juice-shop

Run:

```bash
ssh INTERNAL_IP
```

Example:

```bash
ssh 10.0.2.3
```

If prompted:

```text
Are you sure you want to continue connecting?
```

Type:

```text
yes
```

You are now connected to the `juice-shop` VM.
