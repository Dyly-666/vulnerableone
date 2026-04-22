---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/reconnaissance/active-scanning/wordlist-scanning
---

# Wordlist Scanning

Adversaries may iteratively probe infrastructure using brute-forcing and crawling techniques. Wordlists used in these scans may contain generic, commonly used names and file extensions or terms specific to a particular software. Adversaries may also create custom, target-specific wordlists using data gathered from other Reconnaissance techniques.

Techniques scanning on target domain including:

* [Fuzzing files extension](https://www.computerhope.com/issues/ch001789.htm)
* Fuzzing binary files name
* Fuzzing directory
* Fuzzing parameter
* Fuzzing ID number

Techniques scanning on sub domain including custom wordlist:

* sub-dev.domain.com
* sub-uat.domain.com
* subuat.domain.com
* sub-pre.domain.com
* subdr.domain.com
* sub-api.domain.com

Tools:

* Dirb
* Dirbuster
* Gobuster
* [feroxbuster](https://github.com/epi052/feroxbuster)
* ffuf
