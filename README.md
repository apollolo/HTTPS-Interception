# HTTPS Interception 
**Boston University EC700 Epoch1 Project:** 

**Authors: Muzammil Hussain and Apollo Lo**

Our project involves exploring different anti-virus software's ability to conduct HTTPS interception as well as investigating their method of root certificate generation

## Background

Intercepting HTTPS traffic is a common practice in today's world. It is mostly used by government agency or cooperation to monitor network traffic for the employees. The same technique is being used by commercial anti-virus to analyze the traffic and detect malware for the user. Although it may seem like a reasonable idea to install these anti-viruses as a protection, the harm outweighs the benefit. According to The Security Impact of HTTPS Interception, published in 2017, interception conducted by many anti-virus software have shown to drastically reduce connection security, either by not verifying the certificate or establish weak TLS handshake. 

Whenever the user connects to a website, the anti-virus will terminate and decrypt the client-initiated TLS session and create a TLS session with the destination website. Incoming data from the website are sent to anti-virus to analyze for malware. Once it is deemed safe, then the content is sent to the user in the browser. Since TLS is created with security in mind, it has certificate validation that authenticate the identity of the website and reject imposters. Anti-virus solves this problem by creating signed root certificate in client's computer. Each time the anti-virus intercepts the data, it will dynamically generate a certificate with the website's domain name which will then be approved by the browser.


## Method

Our project aims to verify whether the problems pointed out back in 2017 is still prevalent in 2021. We installed 3 different anti-virus software that have the ability of HTTPS interception, AVG, Avast, and Dr. Web on two different environments, HP Computer with Windows 10, and Oracle Windows VM.

Our first step is to understand how these anti-virus software conduct SSL/TLS handshakes when connected to a website. Some of the problems the paper had were that TLS version was outdated and vulnerable cipher suites were advertised (RC4, DES). The paper also stated that HTTPS interception is easily identifiable from the website/server side. When TLS session is intercepted by anti-virus, it creates another TLS session with the destination. The new TLS session uses a completely different library than web browsers, thus allowing websites to identify mismatched between HTTP User-Agent header and TLS ClientHello.

The next step is to understand how root certificates are generated when anti-virus is installed. We want to see what files are being accessed and created by the anti-virus to generate certificates. The paper identified that some of the anti-virus failed to validate certificates correctly. However, it did not go into details on the method in their experiments. We want to see if we can identify where the certificate key-pair is stored and see if we can replicate the same certificate.

### Wireshark

To check for details in TLS handshake, we installed Wireshark on our devices and monitor the network traffic when anti-virus is on. We look specifically on Client Hello for TLS version and cipher suites. We compare the results with the browser we are using to see if there are mismatch in cipher suites. 

### SysInterval

To understand how root certificates are generated, we want to conduct a dynamic analysis during the installation process since we don't have the source code for these anti-viruses. Using Progress Monitor, we capture what kind of activity are happening and what files are being generated from anti-virus executables. 

### Detect It Easy 

During our project we wanted to conduct static analysis to further understand how certificate generation works. However, we could not find a useful decompiler for the anti-virus executables. Our initial idea was to use dotPeek, a free .NET decompiler created by JetBrains, to decompile anti-virus executables. However, the program yields no results and showed "not supported", this led us to the conclusion that the program is not written in C#. We downloaded an open-source program called Detect It Easy to see if we can find any useful information by analyzing the hex and embedded strings. 



### Findings

From our project, we discover that AVG and Avast are almost identical, not just in their method of root certificate generation, but their UI design is essentially same. We discovered that Avast acquired AVG in July 2016, thus there is a chance that the two anti-virus we investigated have similar source code. 

Also, we were unable to run Avast and AVG on virtual machines, as HTTPS interception does not occur at all. Both anti-viruses do not work for newly installed window on the HP computer as well, until couple weeks later when the HTTPS interception finally began to work. There may be a chance that these two anti-virus software have method of identifying sandboxes and use it to prevent researchers in investigating. Or our method of installation was just wrong, and it only worked later as we cannot identify the exact cause of this behavior. 

With the Dr. Web private key exacted, we were able to use to sign certificates for any websites we want. We tested this method which we will demonstrate in our DEMO. If the attacker has control over the DNS server and the private keys, then it can hijack any website that is being accessed by the victim and present fake website to the victim. 

