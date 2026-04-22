---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/buffer-overflow/bof-tcm/1-spiking
---

# 1- Spiking

**Spiking:** Finding the vulnerable Part of the program.

Run the Vulnserver software with administrator:

<figure><img src="../../../../../../.gitbook/assets/image (615).png" alt=""><figcaption></figcaption></figure>

Run the Immunity Debug software with administrator. Go to **File** and select **Attach** and select **Vulnserver.**

<figure><img src="../../../../../../.gitbook/assets/image (604).png" alt=""><figcaption></figcaption></figure>

We can check the status at the bottom right corner <mark style="color:red;">**"Paused"**</mark>

<figure><img src="../../../../../../.gitbook/assets/image (610).png" alt=""><figcaption></figcaption></figure>

By clicking button **Run** or **F9** and we can check status <mark style="color:green;">**"Running"**</mark> at the bottom right corner.

<figure><img src="../../../../../../.gitbook/assets/image (616).png" alt=""><figcaption></figcaption></figure>

From Kali machine, use **NetCat** tool connect to the vulnserver:

<figure><img src="../../../../../../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

To perform spike, we can use tool "**generic\_send\_tcp**"

<figure><img src="../../../../../../.gitbook/assets/image (237).png" alt=""><figcaption></figcaption></figure>

For **spike\_script**: we send randomly variable to try to break the part of the program **"STATS"**.

<figure><img src="../../../../../../.gitbook/assets/image (240).png" alt=""><figcaption></figcaption></figure>

We start to send out randomly variable to test whether that part of the program vulnerable or not and nothing happened and it doesn't look like vulnerable.

<figure><img src="../../../../../../.gitbook/assets/image (606).png" alt=""><figcaption></figcaption></figure>

Again, we send randomly variable to try to break the part of the program **"TRUN"**.

<figure><img src="../../../../../../.gitbook/assets/image (608).png" alt=""><figcaption></figcaption></figure>

We found out that, after send out randomly to the TRUN part, immediately the immunity debug start blinking. We can check the status and found <mark style="color:red;">**"Paused**</mark>**"** and **Access violation**.

<figure><img src="../../../../../../.gitbook/assets/image (597).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

Also noticed that the vulnserver already **crashed**.

<figure><img src="../../../../../../.gitbook/assets/image (614).png" alt=""><figcaption></figcaption></figure>

On Immunity Debugger, we found that the buffer space was filled out by letter **"A"** and actually it was filled over. Noticed that we have filled over **ESP (Extended Stack Pointer)**, **EBP (Extended Based Pointer)**, and **EIP (Extended Instruction Pointer) / Return address**.

The EIP is the important. The fact, if we can control this **EIP**, we can point this to malicious.

<figure><img src="../../../../../../.gitbook/assets/image (251).png" alt=""><figcaption></figcaption></figure>

