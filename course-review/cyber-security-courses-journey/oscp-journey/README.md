---
hidden: true
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey
---

# OSCP Journey

## Introduction

I want to share my journey while taking [<mark style="color:red;">**Offensive Security Certified Professional (OSCP)**</mark>](https://www.offensive-security.com/pwk-oscp/) with Active Directory Environment. This post is described my journey while studying for OSCP. Everyone will have their own way or experience with Network / Web Application / SystemAdmin skills. So, the preparation and learning path will be various from one to other. I just want to share some resource that could be helpful for other which has no idea what to do before registering for OSCP.

> You will never be ready. Start doing it Now.

## Resource for Preparation

Before enrolling in the OSCP, I played a lot of machines which are the CTF machine's style.

* [<mark style="color:green;">**HackTheBox**</mark>](https://www.hackthebox.com/) - Subscribe 14$/Month
* [<mark style="color:red;">**Proving Ground Play**</mark>](https://www.offensive-security.com/labs/individual/) - Free (daily 3 hours per day)
* [<mark style="color:red;">**Proving Ground Practice**</mark>](https://www.offensive-security.com/labs/individual/) - Subscribe 20$/Month
* [<mark style="color:green;">**TryHackMe**</mark>](https://tryhackme.com/) - Free
* [<mark style="color:orange;">**Vulnhub**</mark> ](https://www.vulnhub.com/)- Free

### Contents

Yes, you're right. You can not crack all of those machines on that platform and you can not go without any information as well. You can check out the below resource:

1.  [**TJ\_Null's**](https://twitter.com/TJ_Null) list of the machines you can work on before jumping into OSCP. He will categorize some of the machines for you. You can copy it to your own excel file and list down which machine you have done so far.\
    [https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=0](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=0)

    <figure><img src="../../../.gitbook/assets/image (695).png" alt=""><figcaption></figcaption></figure>
2.  [**IppSec**](https://twitter.com/ippsec), everyone mostly starts learning by watching IppSec's videos. He didn't just simply show you how to compromise the machine, but he shows you the mistake while compromising the machine. That's really good to know. He always brings new techniques to us.\
    [https://www.youtube.com/@ippsec/featured](https://www.youtube.com/@ippsec/featured)

    <figure><img src="../../../.gitbook/assets/image (638).png" alt=""><figcaption></figcaption></figure>

    There is also a website by IppSec, which could allow you to search for abusing techniques and it will show you the video reference to that.\
    [https://ippsec.rocks/?#](https://ippsec.rocks/?)

    <figure><img src="../../../.gitbook/assets/image (699).png" alt=""><figcaption></figcaption></figure>
3. [**Rana Khalil**](https://twitter.com/rana__khalil) and [**0xdf**](https://twitter.com/0xdf_) blog post on how to compromise those machines which also really help for me to learn different ways to compromise that box.\
   \
   Rana Khalil: [https://rana-khalil.gitbook.io/hack-the-box-oscp-preparation/](https://rana-khalil.gitbook.io/hack-the-box-oscp-preparation/)\
   0xdf: [https://0xdf.gitlab.io/](https://0xdf.gitlab.io/)<br>
4. [<mark style="color:red;">**TCM security**</mark>](https://academy.tcm-sec.com/courses)'s course for [**Windows privilege escalation**](https://academy.tcm-sec.com/p/windows-privilege-escalation-for-beginners) and [**Linux privilege escalation**](https://academy.tcm-sec.com/p/linux-privilege-escalation) provided you with a solid understanding of how to find misconfigured on the system by automating tools and manually.<br>
5. <mark style="color:orange;">**Buffer Overflow**</mark> is a really interesting topic. You can check out the content below from TCM Security and [**Tib3rius**](https://twitter.com/0xTib3rius) provided you with the walkthrough and explained clearly easy steps.\
   \
   **TCM Security**: [https://www.youtube.com/watch?v=qSnPayW6F7U\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G](https://www.youtube.com/watch?v=qSnPayW6F7U\&list=PLLKT__MCUeix3O0DPbmuaRuR_4Hxo4m3G)\
   \
   **Trib3rius**: [https://tryhackme.com/room/bufferoverflowprep](https://tryhackme.com/room/bufferoverflowprep)<br>
6. CheatSheet for an initial scan, enumerate, exploit, and privilege escalation.\
   **HackTricks**: [https://book.hacktricks.xyz/welcome/readme](https://book.hacktricks.xyz/welcome/readme)\
   **PayloadAllTheThings**: [https://github.com/swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)<br>
7. The last one will be the walkthrough from Offensive Security **(S1REN!)**. I really enjoy the way she compromised the box. Especially, the way she shares the tips and note-taking while enumerating the machines.\
   **Youtube**: [https://www.youtube.com/watch?v=NQ6jbKqkJ0s\&list=PLJrSyRNlZ2EeqkJa12Tu-Ezun9kXvHufN](https://www.youtube.com/watch?v=NQ6jbKqkJ0s\&list=PLJrSyRNlZ2EeqkJa12Tu-Ezun9kXvHufN)

You can go through all of the above contents first before enrolling in OSCP if you're not ready. Otherwise, you could register for OSCP directly if you are familiar with that topic.

If you have finished the OSCP course material and labs, but still do not yet feel ready, you can go through the above contents for practice.

> No one could tell which way is better for you. Only you yourself to decide.
>
> Stay in your lane. Run your own race.

## My Progress and Some Tips

I am familiar with the command line on both Linux and Windows by learning day by day by learning from the IppSec video and compromised hack the box machine. Especially, I have already passed the PNPT certificate and that taught me a lot related to the Active Directory concept.

Once I feel ready, I have enrolled in OSCP (2months subscription) by the end of February. I would suggest going through all the course material along with the video. If you do not understand the concept from the text, you can watch it on video. Of course, the lab is important. There are a lot of machines in the lab environment and those machines contain different types of vulnerability which enabled us to learn of various attack paths.

Don't miss that PWK lab. Of course the Active Directory Part. Due to an exam, you have to encounter the Active Directory environment which will be 40 Points (Fully compromised). If you can't compromise that AD environment, you will have less chance to pass.

> <mark style="color:red;">**Why Active Directory part is important?**</mark>

I will share this information to calculate your points for a passing score of **70/100**.

#### \* Bonus 10 Points

<figure><img src="../../../.gitbook/assets/image (662).png" alt=""><figcaption></figcaption></figure>

If you have completed the exercise above, you will learn a lot from that exercise and lab machines and of course, you will get **10 points** bonus in advance.

<figure><img src="../../../.gitbook/assets/image (642).png" alt=""><figcaption></figcaption></figure>

As you can see, option number 4 will be the best one for us. You just need to compromise the AD environment in the exam and root 1 of the machines, you will secure your 70 points.

No need to be rushed, otherwise you will miss a lot of techniques that have been taught in PWK material and Labs.

While compromising each of the machines, I suggest writing down the attack path with an explanation like you're doing reporting. Yes right, it will take time. But it will help you a lot on exams, you will not miss your screenshot or feel nervous about the reporting part.

> Report is the critical part of penetration testing process. It provides detailed information about the security issue and remediation part as well for Executive and Technical person.

You can check out my **HackTheBox** walkthrough. No need to follow my path, just get some ideas and generate your style of reporting.

<figure><img src="../../../.gitbook/assets/image (646).png" alt=""><figcaption></figcaption></figure>

**HackTheBox Walkthrough:**

{% embed url="https://vulnableone.gitbook.io/vulnableone/course-review/cyber-security-courses-journey/oscp-journey/ctf/hack-the-box/" %}

## #1 Attempted Failure

I failed my first attempt as I was confused about the Time Zone. I have booked at **11 AM**, but actually, my exam time should be **6 PM**. It really messes up my preparation feeling :joy:. Make sure you check your Time Zone and exam Time Zone properly.

I have to spend around **6 hours (12 AM)** to enumerate the Active directory but still not getting the right path to foothold. I decided to sleep (feeling given up already). I wake up at **6 AM** to freshen up and start enumerating again. Suddenly I found the foothold right in front of me. <mark style="color:red;">**"I'm thinking way too far"**</mark>.

I wrote down the attack path and pause for having breakfast and a cup of Café to boost my energy. By coming back, I can perform the lateral movement to the compromised full Active Directory.

For stand-alone machines, I could gain only 1 foothold on the machines. I could not perform privilege escalation on the machine.

Till the time rut out, I only compromised the full Active directory with (40 Points) and foothold on the machine (10 Points) plus my bonus 10 points. Totally, I just got 60 points which are really close to the passing score.

## #2 Attempted Success

By my first attempt, I realized I was so rushed for the exam. I just started the exam once I completed my Lab time. So that my attack path will be less.\
\
I decide to subscribe to **Proving Ground for 1 month** and watch and learn from **S1REN!** walkthrough from Offensive Security. I realized that I have missed a lot of information such as note-keeping and techniques for quick calling commands to run and enumeration.

Again unplanned exam, after learning for the whole month, I think nothing is left for me to do. By just buying the voucher for another attempt, suddenly I saw scheduled on **Saturday at 10 AM** which is a free time slot. I decided to book the exam immediately.

On my 2nd attempt, it just got harder than my first time :joy: <mark style="color:red;">**Again feeling like giving up**</mark> :joy: By Trying Harder, I spent more than 10 hours in order to find a foothold in Active Directory Environment. This time I decide not to sleep. I go straight from a foothold and lateral movement to compromise the full Active Directory.

For my first stand-alone machines, it just took me a few hours in order to fully compromise the machine. Now I have secured 60 points.

Unfortunately, I could not compromise the remaining machine. I decided to stop that and collect Proof of Concept for my reporting due to it just 2hours left for my exam time.

Once I've done my report part, I decided to revert those remaining machines. Then, my exploit worked !!! I realized that maybe my scanning process or I might have done something wrong to cause the machine error. Now, I have a secure passing score of **70 points.**

> <mark style="color:red;">**I would suggest not to thinking way too far. You have limit of 24 reverts on exam environment. Make sure to use it if you feeling you're going on the right path.**</mark>

Finally, I have submitted my report. It just took 1days to get my results back. :tada::heart\_eyes::tada:

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## Conclusion

You just started not ended !!!

By completing this course, you could have a lot of techniques for Penetration Testing when encountering **Strange Service** or **Strange Environment**, you will find a way to test it manually. You have concepts on how to find a vulnerability and smell the chain of attack once you see the misconfiguration.

Keep learning and Sharing back to the newcomer and the community.

Make sure to read the Exam Guide:

[https://help.offensive-security.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide](https://help.offensive-security.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide)
