# Privilege Escalation using the Dirty Cow Kernel Exploit
<!-- hide -->

> By [@rosinni](https://github.com/rosinni) and [other contributors](https://github.com/breatheco-de/kernel-exploit-dirtycow-project/graphs/contributors) at [4Geeks Academy](https://4geeksacademy.co/)

[![build by developers](https://img.shields.io/badge/build_by-Developers-blue)](https://4geeks.com)
[![build by developers](https://img.shields.io/twitter/follow/4geeksacademy?style=social&logo=twitter)](https://twitter.com/4geeksacademy)

*These instructions are also [available in Spanish](https://github.com/breatheco-de/kernel-exploit-dirtycow-project/blob/main/README.es.md)*
<!-- endhide -->

<!-- hide -->

### Before you begin...

> We need you! These exercises are created and maintained in collaboration with people like you. If you find any errors or typos, please contribute and/or report them.

<!-- endhide -->

## 🌱 How to start this project?

The academy provides a virtual machine with a vulnerable version of Ubuntu Server (16.04.1), running a kernel affected by the **Dirty Cow** vulnerability (CVE-2016-5195).

As a student, you already have access to this system with a limited user called `student`. Your goal will be to **identify that the system is vulnerable**, **compile and run a real exploit**, and **escalate your privileges to gain root access**. This exercise simulates a realistic scenario where a local attacker, without administrative privileges, fully compromises the system by exploiting a kernel vulnerability. The objectives are:

- Check the kernel version of a Linux system.
- Compile a real exploit using `g++`, a compiler used to build and link programs written in **C++**, generating an executable from the source code.
- Escalate privileges from a limited user to `root`.
- Demonstrate the success of the attack by capturing a flag located in `/root`.

This type of exploitation is typical in advanced security audits and Red Team environments. It will help you connect low-level operating system concepts with real offensive techniques, in a practical and guided way.

### Requirements

* [Vulnerable Ubuntu machine with kernel `4.4.0-21-generic` (or similar unpatched)](https://storage.cloud.google.com/cybersecurity-machines/dirty-cow-lab.ova)
* Access as a non-privileged user on the vulnerable machine (`student:password123`)
* Kali Linux machine **(Attacker)**. This is the machine where you will prepare the exploit, and it must have the following tools installed:

    - **🔧 `Docker`:** We will use Docker to launch an Ubuntu 16.04 container and compile the exploit with the same libraries as the victim machine. This ensures compatibility and avoids errors due to modern compiler or `glibc` versions.

    - **🔧 `g++`:** This is the C++ compiler. We will use it to compile the `dirty.cpp` exploit, which is written in this language. It allows us to generate an executable (`dirty`) from the source code.

    - **🔧 `scp` (Secure Copy Protocol):** This is a tool for securely copying files between Linux systems. We will use it to transfer the compiled exploit from Kali to the victim machine.



> 🚨 Legal Notice: This repository contains a clean and commented version of the **Dirty Cow** exploit (CVE-2016-5195), designed exclusively for **educational** purposes. This variant creates a new user with root privileges by exploiting a race condition in the Linux kernel memory subsystem. This exploit is for **educational use only**. It must be used in **controlled and legal** environments, such as labs, practice virtual machines, or cybersecurity classes.

## 📝 Instructions

### Step 1: Check the kernel version

1. Log in to the vulnerable machine with the limited user and run:

    ```bash
    uname -a
    ```

    This returns something like:

    ```bash
    Linux dirtycow-lab 4.4.0-31-generic #50-Ubuntu SMP Tue Sep 6 15:42:33 UTC 2016 x86_64 x86_64 x86_64 
    ```
2. Note the version and research how to exploit that vulnerability using databases like [Exploit-DB](https://www.exploit-db.com/exploits/40847), GitHub, or searchsploit from Kali. For this exercise, **we provide you with a functional and documented exploit that you can [download here](https://raw.githubusercontent.com/breatheco-de/kernel-exploit-dirtycow-project/refs/heads/main/assets/dirty.cpp)**.



> ⚠️ **IMPORTANT!** The vulnerable machine does not have compilation tools installed, nor `sudo` permissions to add them. Therefore, we must compile the exploit on Kali, inside a Docker container with Ubuntu 16.04, which has the same versions of glibc, libstdc++, and system libraries as the vulnerable machine; otherwise, you may encounter version issues.

### Step 2: Prepare the environment in Kali with Docker

1. Install Docker on Kali:

    ```bash
    sudo apt update
    sudo apt install docker.io -y
    sudo systemctl start docker
    sudo systemctl enable docker
    ```

2. Download the Ubuntu 16.04 image:

    ```bash
    sudo docker pull ubuntu:16.04
    ```

3. Launch a container:

    ```bash
    sudo docker run -it --name compile-ubuntu16 ubuntu:16.04
    ```

4. Install compilation tools:

    ```bash
    apt update
    apt install build-essential libutil-dev -y
    ```

### Step 3: Create and compile the dirty.cpp exploit

1. Create the file inside the container:

    ```bash
    nano dirty.cpp
    ```

    (Paste the full exploit code from [dirty.cpp](https://raw.githubusercontent.com/breatheco-de/kernel-exploit-dirtycow-project/refs/heads/main/assets/dirty.cpp) provided by the academy or from [Exploit-DB 40847](https://www.exploit-db.com/exploits/40847))

2. Compile the binary:

    ```bash
    g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dirty dirty.cpp -lutil
    ```

3. Exit the container:

    ```bash
    exit
    ```

4. Copy the compiled binary from the container to Kali:

    ```bash
    sudo docker cp compile-ubuntu16:/dirty ./dirty
    ```

### Step 4: Run on the victim

1. Transfer the binary to the victim:

    ```bash
    scp dirty student@<VICTIM_IP>:/home/student
    ```

2. On the victim, run it:

    ```bash
    chmod +x dirty
    ./dirty
    ```

    If the exploit is successful, you will see a message telling you the password assigned to the root user.

### Step 5: Escalate privileges

1. Once the exploit finishes executing, switch to the `root` user.
2. Verify that you have root access by entering the generated password.
3. Finally, capture the flag. To confirm the success of the attack, read the contents of the flag file located in the `/root/flag.txt` directory. If everything worked correctly, you will see the flag's content.

