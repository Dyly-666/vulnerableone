---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/buffer-overflow/bof-tcm
---

# BOF - TCM

1. **Spiking:** Finding the vulnerable Part of the program.
2. **Fuzzing:** Sending a bunch of characters at a program and see if we can break it.
3. **Finding the Offset:** Find out what point we did break it. We want to find something called the offset.
4. **Overwriting the EIP:** We use that offset to overwrite the EIP that pointer address.
5. **Finding Bad Characters:** Once we have EIP controlled, we need to cleanup thing that is called Finding Bad Characters or Finding the right module.
6. **Finding the Right Module:** After bad characters, we have to find the right module.
7. **Generating Shellcode:** Once we have the information from step 5 to 6, we can generate the malicious shell code that will allow us to get the reverse shell.

