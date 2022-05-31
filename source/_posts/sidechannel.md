title: A review on papers of RowHammer, Specture and HeartBleed
date: 2022/05/01 16:10:00
categories:
- PaperReview
tags:
- Security
- Research
thumbnail: https://raw.githubusercontent.com/breakertt/blog/pic/meltdown-spectre-kernel-vulnerability.png
toc: true
---

Litrature review on RowHammer, Specture and HeartBleed

<!-- more -->

# Summary

## Flipping Bits in Memory Without Accessing Them: An Experimental Study of DRAM Disturbance Error

As the DRAM processing technology continuously evolves, the DRAM cells are considered to be more likely to be interference by other cells. Kim et al. [1] discovered a vulnerability in modern DRAM chips, named Rowhammer attack afterwards. A Rowhammer attack can flipbits in a row on a DRAM chip by frequent accessing another physical adjacent row.

The authors firstly demonstrated this vulnerability can be easily exploited with a few lines of assembly on a commodity PC. Then in-depth analysis with an FPGA-based testing platform is performed to study behaviour and root cause. It is concluded that most DRAM modules manufactured in 2012 and 2013 are affected and the number of accesses required is relatively small. Finally, a novel mitigation solution is proposed.

## The Matter of Heartbleed

The Heartbleed vulnerability is a software bug in a widely used and fundamental cryptography library OpenSSL and consequently, is considered to be a serious security issue once disclosed. Based on such a phenomenal vulnerability, the actual impact of the vulnerability and responses from various sides including bug patching, certificaterevoking and attacks before and after disclosure are measured and evaluated in this paper [2]. The investigation shows the impact of this vulnerability is indeed great with massive attacks. Many sites are patched, but most of them are patched incorrectly (no certificate replacement). Additionally, the authors notifiedthe administrators of unpatched services to study the effectiveness of post-disclosure community mitigation measures and it is confirmedto be useful.

Building upon the previous investigation and experiment, five aspects are discussed with weak- nesses and strengths observed and concrete suggestions.

## Spectre Attacks: Exploiting Speculative Execution

The Spectre is a well-known side-channel attack proposed by Kocher et al. [3] This attack exploits micro-architectural features including speculative execution, branch prediction and caches which are common in modern computer processors.

The Spectre attack can be performed in three phases. The branch prediction unit (BPU) is trained maliciously for exploitable instructions to be speculatively executed. Then, the instructions are modifiedto ones perform unexpected accesses and fail the branch condition. The instructions will be executed speculatively with mis-trained processor as a micro-architectural behaviour and leave cache-based covert channels. Then the attacker can retrieve secret data from the channel. Based on this procedure, two variants are discussed where poisons branch predictor and branch target buffer (BTB) respectively.

It is demonstrated nearly all processor vendors are affected, and the software security systems relying on hardware security guarantees are consequently becoming vulnerable. And various methods are suggested to prevent the Spectre attack.

# Key Themes

A typical structure is observed in these three papers, which study or even reveal famous failures. The vulnerability and the attack around it are explained as a starting point. Then the range and impact of the attack are well studied. Potential solutions are proposed in the end, but to different extents, depending on the nature of vulnerability. It is worth noting that Kim et al. and Kocher et al. attaches more importance to the explanation as they are the papers that reveal the attack, while Durumeric et al. focus on the impact. Under such structure, the reader can obtain a clear understanding of attack strategy and root cause (what), the impact (why) and the solutions (how).

What the vulnerability is? I would consider all three attacks as side-channel, following the definition in Ross Anderson’s book - "A side channel is where information leaks accidentally via some medium that was not designed or intended for communication". It may be more intuitive for the Spectre and the Heartbleed as they enable the read privilege for the attacker. Although attacks like the RowHammer don’t leak data directly, an attacker can change the permissions bits in the memory to enable the read privilege. Therefore, the attacker can get control over the system regardless of the direct privilege provided by the attack. However, the nature of the side-channels implied is diverse. The HeartBleed is a pure software bug, the Spectre is a vulnerability on the central hardware unit (CPU), and the RowHammer exploits a relative peripheral hardware unit (Memory Modules).

Why we should care about this vulnerability? Although all three attacks are famous, the impact varies on factors like exploitation difficulty and influence range. Since the HeartBleed attack is a web software failure, an attacker can exploit it remotely. By contrast, the other two attacks are more challenging to perform as the attacker must run some program on the machine. Furthermore, it is non-trivial to precisely manipulate the adjacent memory rows around the row with critical data with several levels of memory address translation from operating system, memory management unit, etc., making the Rowhammer the most challenging one to exploit. However, the hardware ones may be more severe in terms of influencerange, as the number of vulnerable websites to the HeartBleed is much less than vulnerable processors and memory modules.

How to Mitigate or Solve it? It is also found that fixing a software-based side-channel like the HeartBleed is the most straightforward. Although there are software techniques to mitigate the hardware-based side-channels, they are not a long-term solution and blur the hardware’s responsibili- ties. Thus, it is essential to solve fundamentally from the hardware side. Unfortunately, It may take months or even years for vendors to re-design and ship. The fact that consumers have little incentive to pay for the newest patched hardware make the patch cycle even longer. It is agreed by the authors of the Spectre and the Rowhammer that hardware vendors must take care of the trade-off between performance and security. However, when considering the fixitself, the low certificaterevocation rate in the HeartBleed case indicated a gap between a patch and a correct patch. Since the level of software users is unaligned, the correct patch rate is significantlylower than the patch rate. Meanwhile, the correct path rate of hardware vulnerability may be higher.

With intensive comparison among three side-channel attacks, it is interesting that software-based side-channel vulnerability is easy to exploit but quick to patch. However, a hardware one is non-trivial to exploit but leaves long-term pain. Again, it is not practical to determine which one is more severe without a certain threat model.

# Legacy

(Attack evolution) A clear evolution trend on the hardware side-channel is noticed from the RowHam- mer (2014) to the Spectre (2019), then today (2022). Enhancing existing side-channel attacks are intuitive. There are attempts on extending existing attacks to more platforms and more applications. For example, the Drammer attack [4] demonstrates the possibility of the Rowhammer attack on mobile platforms, and the ZombieLoad attack [5] shows that even enclaves are vulnerable to transient attacks. Efforts are also made to decrease the exploitation difficultyof side-channels by introducing remote execution and automated attack features into the attack. The Rowhammer.js [6] and the NetSpectre [7] are presented as the remote variant of the Rowhammer and the Spectre. The Rowhammer.js attack and the cache templates attacks [8] on cache side-channels can both be performed automatically. Apart from the variants, attacks on different side-channels are continuously disclosed by research community from memory attack like (the RowHammer [1]), to cache attack like Flush-and-Flush [9], then finally, out-of-out execution units for the Meltdown [10] and branch prediction units for Spectre [3] The side-channel is becoming easier to exploit but harder to be solved once-for-all [11]

(Patch issues) The security community was recently shocked by the disclosure of a bug in log4j, a similar software side-channel to the HeartBleed [2] Systematic risk in the modern software world where software heavily relies on libraries is indicated, thus, bringing the importance of fast reaction ability and prevention mechanisms. Li and Paxson [12] provided valuable suggestions after a large- scale study on the vulnerabilities and corresponding patches of 682 open-source software projects, rather than focusing on only one vulnerability like the Heartbleed. In terms of tools, an Internet-wide scanning tool played a vital role in studying the Heartbleed matter. Later on, Durumeric et al. [13] developed a free and democratized version of this tool to help discover vulnerable web services deployments, significantlyreducing the cost of real-time Internet-wide scan. Another growing concern is overlooking future side-channel variants in mitigation methods (i.e. patches). Canella et al. [11] showed the origin defence methods to the Spectre and the Meltdown can be trivially bypassed with naive variants. Therefore, building a fixingplan and trade-off between quick but weak solutions and slow but robust solutions is essential.

# References

1. Yoongu Kim, Ross Daly, Jeremie Kim, Chris Fallin, Ji Hye Lee, Donghyuk Lee, Chris Wilkerson, Konrad Lai, and Onur Mutlu. Flipping bits in memory without accessing them: An experimental study of DRAM disturbance errors. In 2014 ACM/IEEE 41st International Symposium on Computer Architecture (ISCA), pages 361–372, Minneapolis, MN, USA, June 2014. IEEE.
1. Zakir Durumeric, Frank Li, James Kasten, Johanna Amann, Jethro Beekman, Mathias Payer, Nicolas Weaver, David Adrian, Vern Paxson, Michael Bailey, and J. Alex Halderman. The Matter of Heartbleed. In Proceedings of the 2014 Conference on Internet Measurement Conference, pages 475–488, Vancouver BC Canada, November 2014. ACM.
1. Paul Kocher, Jann Horn, Anders Fogh, Daniel Genkin, Daniel Gruss, Werner Haas, Mike Hamburg, Moritz Lipp, Stefan Mangard, Thomas Prescher, Michael Schwarz, and Yuval Yarom. Spectre Attacks: Exploiting Speculative Execution. In 2019 IEEE Symposium on Security and Privacy (SP), pages 1–19, 2019. ISSN: 2375-1207.
1. Victor van der Veen, Yanick Fratantonio, Martina Lindorfer, Daniel Gruss, Clementine Maurice, Giovanni Vigna, Herbert Bos, Kaveh Razavi, and Cristiano Giuffrida. Drammer: Deterministic Rowhammer Attacks on Mobile Platforms. In Proceedings of the 2016 ACM SIGSAC Conference on Computer and Communications Security, CCS ’16, pages 1675–1689, New York, NY, USA, 2016. Association for Computing Machinery.
1. Michael Schwarz, Moritz Lipp, Daniel Moghimi, Jo Van Bulck, Julian Stecklina, Thomas Prescher, and Daniel Gruss. ZombieLoad: Cross-Privilege-Boundary Data Sampling. InProceedings of the 2019 ACM SIGSAC Conference on Computer and Communications Security, CCS ’19, pages 753–768, New York, NY, USA, 2019. Association for Computing Machinery.
6. Daniel Gruss, Clémentine Maurice, and Stefan Mangard. Rowhammer.js: A Remote Software-Induced Fault Attack in JavaScript. In Juan Caballero, Urko Zurutuza, and Ricardo J. Rodríguez, editors,Detection of Intrusions and Malware, and Vulnerability Assessment, Lecture Notes in Computer Science, pages 300–321, Cham, 2016. Springer International Publishing.
6. Michael Schwarz, Martin Schwarzl, Moritz Lipp, Jon Masters, and Daniel Gruss. NetSpectre: Read Arbitrary Memory over Network. In Kazue Sako, Steve Schneider, and Peter Y. A. Ryan, editors, Computer Security – ESORICS 2019, Lecture Notes in Computer Science, pages 279–299, Cham, 2019. Springer International Publishing.
6. Daniel Gruss, Raphael Spreitzer, and Stefan Mangard. Cache Template Attacks: Automating Attacks on Inclusive {Last-Level} Caches. pages 897–912, 2015.
6. Daniel Gruss, Clémentine Maurice, Klaus Wagner, and Stefan Mangard. Flush+Flush: A Fast and Stealthy Cache Attack. In Juan Caballero, Urko Zurutuza, and Ricardo J. Rodríguez, editors, Detection of Intrusions and Malware, and Vulnerability Assessment, Lecture Notes in Computer Science, pages 279–299, Cham, 2016. Springer International Publishing.
6. Moritz Lipp, Michael Schwarz, Daniel Gruss, Thomas Prescher, Werner Haas, Anders Fogh, Jann Horn, Stefan Mangard, Paul Kocher, Daniel Genkin, et al. Meltdown: Reading kernel memory from user space. In 27th USENIX Security Symposium (USENIX Security 18), pages 973–990, 2018.
6. Claudio Canella, Jo Van Bulck, Michael Schwarz, Moritz Lipp, Benjamin von Berg, Philipp Ortner, Frank Piessens, Dmitry Evtyushkin, and Daniel Gruss. A Systematic Evaluation of Transient Execution Attacks and Defenses. pages 249–266, 2019.
6. Frank Li and Vern Paxson. A Large-Scale Empirical Study of Security Patches. In Proceedings of the 2017 ACM SIGSAC Conference on Computer and Communications Security, CCS ’17, pages 2201–2215, New York, NY, USA, 2017. Association for Computing Machinery.
6. Zakir Durumeric, David Adrian, Ariana Mirian, Michael Bailey, and J. Alex Halderman. A Search Engine Backed by Internet-Wide Scanning. In Proceedings of the 22nd ACM SIGSAC Conference on Computer and Communications Security, CCS ’15, pages 542–553, New York, NY, USA, 2015. Association for Computing Machinery.
