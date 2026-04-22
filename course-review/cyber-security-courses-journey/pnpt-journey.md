---
hidden: true
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/pnpt-journey
---

# PNPT Journey

PNPT Exam:

{% embed url="https://certifications.tcm-sec.com/pnpt/?ref=sokpech" %}

In early October 2021, I passed my first practical penetration testing exam on TCM Security’s Certification, PNPT <mark style="color:red;background-color:red;">**(Practical Network Penetration Tester)**</mark>. Since the exam is interesting and different from other platforms, you need to perform the full scope of penetration testing and especially the **Debrief** part. So, I would like to share my experience with this exam and courses.

## Introduction

Before releasing the PNPT exam and courses, I had learned from **The Cyber Mentor** channel and there is a series of videos called <mark style="color:red;">**"Zero to Hero Pentesting"**</mark>.

Youtube playlist: [https://www.youtube.com/watch?v=qlK174d\_uu8\&list=PLLKT\_\_MCUeiwBa7d7F\_vN1GUwz\_2TmVQj](https://www.youtube.com/watch?v=qlK174d_uu8\&list=PLLKT__MCUeiwBa7d7F_vN1GUwz_2TmVQj)

There are two more series I want to mention that is really helpful for me as I'm looking for free resource and well-explained at that time.

<mark style="color:red;">**Pentetsing for n00bs:**</mark> This series is about some walkthroughs of HTB machines and the methodology of how to compromise those machines.

Youtube Playlist: [https://www.youtube.com/watch?v=JZN3JhoAdWo\&list=PLLKT\_\_MCUeiyxF54dBIkzEXT7h8NgqQUB](https://www.youtube.com/watch?v=JZN3JhoAdWo\&list=PLLKT__MCUeiyxF54dBIkzEXT7h8NgqQUB)

<mark style="color:red;">**Buffer Overflows Made Easy:**</mark> This one is explaining about Buffer Overflow vulnerability. He's sharing some python scripts and step by step to exploit.

Youtube Playlist: [https://www.youtube.com/watch?v=qSnPayW6F7U\&list=PLLKT\_\_MCUeix3O0DPbmuaRuR\_4Hxo4m3G](https://www.youtube.com/watch?v=qSnPayW6F7U\&list=PLLKT__MCUeix3O0DPbmuaRuR_4Hxo4m3G)

## About Course

By 2021, I've seen some people doing reviews on the PNPT exam and courses. Without any further, I've enrolled in PNPT Exam with training as it contained 5 courses.

* [Practical Ethical Hacking](https://academy.tcm-sec.com/p/practical-ethical-hacking-the-complete-course)
* [Linux Privilege Escalation for Beginners](https://academy.tcm-sec.com/p/linux-privilege-escalation)
* [Windows Privilege Escalation for Beginners](https://academy.tcm-sec.com/p/windows-privilege-escalation-for-beginners)
* [Open Source Intelligence (OSINT) Fundamentals](https://academy.tcm-sec.com/p/osint-fundamentals)
* [External Pentest Playbook](https://academy.tcm-sec.com/p/external-pentest-playbook)

<figure><img src="../../.gitbook/assets/image (637).png" alt=""><figcaption></figcaption></figure>

**PEH:** Since I have followed his youtube channel, I'm familiar with course content but still there are new topics and content updated as well.

**Windows / Linux Privilege Escalation:** This one not just prepared me for PNPT, but it help me a lot for my future course as I have concepts from the courses.

**OSINT:** Sock Puppets? This one is my favorite course and I realized the internet is a scary place :joy:

**External Pentest Playbook:** This course had been defined with a clear objective for external attack infrastructure.

* <mark style="color:red;">"Low chance of RCE, High chance of weak passwords"</mark>
* <mark style="color:red;">"Don't start web app assessment when you're focusing on external infrastructure"</mark>
* <mark style="color:red;">"If you find XSS, how are you gonna branch into the internal network??"</mark>
* <mark style="color:red;">"If you see the login portal, you could try SQL injection to see if you can breach the website."</mark>

After completing those 5 courses, I enrolled in extra one as [<mark style="color:red;">**Movement, Pivoting, and Persistence**</mark>](https://academy.tcm-sec.com/p/movement-pivoting-and-persistence-for-pentesters-and-ethical-hackers). That one contained a C2 framework, Email Phishing, Port Forwarding, and more.

All of the courses are affordable at just **29.99$** and they always offer Coupon discounts. Anyway, if you still thinking about the price, he also shares the course content on his youtube channel as well.

Youtube Channel: [https://www.youtube.com/@TCMSecurityAcademy/featured](https://www.youtube.com/@TCMSecurityAcademy/featured)

<figure><img src="../../.gitbook/assets/image (625).png" alt=""><figcaption></figcaption></figure>

## About Exam

<figure><img src="../../.gitbook/assets/image (650).png" alt=""><figcaption></figcaption></figure>

People don't have experience in doing penetration testing. You could follow along with those courses. As they have mentioned 2 options for you.

On exam day, I received a VPN package and Role of Engagement from TCM.

<figure><img src="../../.gitbook/assets/image (682).png" alt=""><figcaption></figcaption></figure>

As we have 5 full days to perform penetration testing on the exam environment and 2 days for doing report. By this, you don't need to stay awake and stress out about the time.

On the engagement letter, we have a Penetration Test scope and some actions are out of scope which clearly defines the objective.

As I couldn't expose what is going on in the exam but it's the same as what they mentioned.

* Perform OSINT to gather information on the target
* Perform external penetration testing
* Leveraging into network, lateral and vertical network movement
* Ultimately compromise the exam domain controller

Then we have to do a report on all finding items and submit the report via email. Next, we will receive a link to schedule our debrief with Heath.

<figure><img src="../../.gitbook/assets/image (651).png" alt=""><figcaption></figcaption></figure>

Finally, the debriefing part for 15 minutes with Heath on finding items. It was really good for me as it was my first time debriefing my report and he also gave advice on Reporting style and debriefing as well.

Once you have cleared all the processes, he will generate the certificate for you and invite you to a private group on discord for PNPT candidates.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## Conclusion

I might share some advice for those who don't have experience and want to take the exam:

* I would recommend completing the OSINT and External Pentest Courses.
* Read the letter of engagement and pay attention.
* Make sure you understand the Active Directory attack path in the course.
* Try not to overthink or think about it in a complicated way.
* Try to look for "how to **Access** it rather than how to **Exploit** it".
* Don't forget Pivoting Tools and Techniques

> You will never be ready. Just Start it.

For those who are looking to start up your penetration testing career, you can check out the TCM Security content. It's really helpful for you as it has a fully completed penetration testing process from end to end. Even though you have failed, they offer you a free retake as well.

As this course and exam is a Real-World penetration testing process. After completing this, you will have a lot of concepts on performing pentest. Of course, the price is affordable rather than any other cyber security platform.
