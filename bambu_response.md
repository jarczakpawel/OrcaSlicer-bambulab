# My response to Bambu Lab's claims

This document explains my position regarding Bambu Lab's accusations about my OrcaSlicer-bambulab work, including the claims about impersonation, reverse engineering, Terms of Use, and alleged bypassing of security measures.

My position is simple: I worked from public AGPL-3.0 source code released by Bambu Lab itself. I did not redistribute Bambu Lab's proprietary `bambu_networking` binaries. I did not reverse-engineer the closed `bambu_networking` plugin. The path I used was based on the public Linux path already present in Bambu Studio's AGPL source code.

This document is separate from the detailed AGPL analysis of `bambu_networking`, but the two issues are closely connected.

---

## 1. What I did and what I did not do

I worked on Bambu Studio / OrcaSlicer code that was publicly released under AGPL-3.0.

I did not redistribute the closed `bambu_networking` plugin binaries.

I did not include Bambu Lab's proprietary network plugin in my repository.

I did not decompile the closed plugin.

I did not patch the closed plugin.

I did not extract hidden secrets from the closed plugin.

I did not need to reverse-engineer the closed plugin, because the relevant integration path is already visible in Bambu Studio's public AGPL source code.

The disputed path was built from public AGPL-covered Bambu Studio code and my own integration work around that code.

---

## 2. Why WSL2 / Lima were used

I have seen claims and reports suggesting that the use of WSL2 was treated as evidence of reverse engineering or bypassing security. I cannot confirm every report as an official statement from Bambu Lab, because after the Reddit conversation I did not continue direct discussions with them. However, I can clearly explain why WSL2 on Windows and Lima on macOS were used.

The reason was simple: the Linux path already existed in the public AGPL Bambu Studio code, and that Linux path worked.

Using WSL2 / Lima made it possible to reuse the existing Linux-side path without touching the closed plugin binary, without patching it, and without reverse-engineering it.

If I had been reverse-engineering the proprietary plugin, it would make no sense to impose the extra complexity of WSL2 / Lima just to run the existing Linux path. The entire point was the opposite: to stay on the public AGPL source-code side enitrely.

This was not a method for bypassing a security system. It was a way to reuse code that Bambu Lab itself had published under AGPL.

---

## 3. Why reverse engineering the plugin was unnecessary

The most basic question is: why would I reverse-engineer `bambu_networking` when Bambu Studio's public AGPL source code already shows how Bambu Studio integrates with it?

The public AGPL source defines the library name, the agent name, the agent version, callback types, shared structures, and the functions that are dynamically resolved from the plugin.

For example, Bambu Studio's public header defines the plugin library and agent information here:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/bambu_networking.hpp#L103-L106

The same public header defines C++ callbacks used across the boundary between Bambu Studio and the plugin:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/bambu_networking.hpp#L123-L145

The public source also shows Bambu Studio dynamically loading `bambu_networking.dll`, `libbambu_networking.so`, or `libbambu_networking.dylib`:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/NetworkAgent.cpp#L184-L281

And it shows Bambu Studio resolving many `bambu_network_*` functions from the plugin:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/NetworkAgent.cpp#L284-L392

These facts are all in public AGPL source code. No reverse engineering of the proprietary plugin is needed to see this architecture.

---

## 4. User-Agent is not authentication

I already addressed Bambu Lab's public User-Agent claims in the README, so I will not repeat the full response here.

The important point is simple: the User-Agent construction comes directly from Bambu Studio's public AGPL source code.

Bambu Studio sets the User-Agent here:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/Http.cpp#L175

The values are defined here:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/version.inc

User-Agent is not a password, token, cryptographic signature, or real authorization mechanism. It is self-declared HTTP client metadata. Any HTTP client can set a User-Agent.

I did not obtain this value from a proprietary binary. I did not extract it by reverse engineering. It was not hidden. It came from public AGPL source code published by Bambu Lab.

---

## ❗ 5. The strongest technical issue: the closed plugin is loaded into the AGPL program ❗

> This is the central point: `bambu_networking` is not merely a file placed next to Bambu Studio. Bambu Studio loads the closed `.dll`, `.so`, or `.dylib` into the AGPL application's own process, resolves functions from that closed module, stores them as function pointers, and executes them inside the AGPL program's runtime.
>
> This is not a neutral external command-line tool. This is closed object code being loaded into and called from the AGPL application process.

The loading code is here:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/NetworkAgent.cpp#L184-L281

After loading the module, Bambu Studio resolves functions from the closed plugin here:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/NetworkAgent.cpp#L284-L392

This is not one small helper function. `NetworkAgent.cpp` resolves a large set of `bambu_network_*` functions, including functions for user login, server connection, printer connection, cloud print, LAN print, sending G-code to SD card, camera URLs, printer firmware, user presets, filament functions, telemetry, HMS snapshots, publishing workflows, and other networking features.

Technically, this means that Bambu Studio takes functions from the closed runtime module and executes them as part of the AGPL application's runtime. The closed plugin is not only "compatible" with Bambu Studio. It is dynamically loaded by Bambu Studio and called through an ABI designed in Bambu Studio's public source code.

That is why calling it "just a plugin" does not solve the AGPL issue. The legal question is not the marketing name of the component. The legal question is how the component actually interacts with the AGPL program.

---

## ❗ 6. Callbacks show control flow between the closed plugin and the AGPL program ❗

The public header `bambu_networking.hpp` defines C++ callbacks that cross the boundary between Bambu Studio and the closed plugin:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/bambu_networking.hpp#L123-L145

These include callbacks for login state, printer connection, local connection, server connection, device messages, HTTP errors, update status, cancellation, waiting for jobs, message arrival, and main-thread scheduling.

The most important type is:

```cpp
// from bambu_networking.hpp
typedef std::function<void(std::function<void()>)> QueueOnMainFn;
```

This is not a simple string. This is not a command-line parameter. This is not a file-based exchange. This is a C++ callback mechanism that allows the closed plugin to schedule work back onto Bambu Studio's main UI thread.

The symbol for this function is resolved from the closed plugin here:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/NetworkAgent.cpp#L304

And `NetworkAgent::set_queue_on_main_fn()` passes the callback to the plugin here:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/NetworkAgent.cpp#L796-L801

This is very strong evidence of control flow between the AGPL program and the closed dynamic component.

In plain terms: the closed plugin is not isolated. It receives function callbacks from the AGPL application, and the AGPL application gives it a path to schedule code execution back into the main UI thread.

That is exactly the type of deep runtime integration that cannot be honestly described as a loose, independent add-on.

---

## 7. Shared data structures show intimate data communication

The public AGPL header also defines shared C++ structures used by the plugin.

The most important example is `PrintParams`:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/bambu_networking.hpp#L197-L247

`PrintParams` contains printer identifiers, task names, project names, file names, FTP fields, MD5, nozzle mappings, AMS mappings, connection type, printer IP, SSL flags, username, password, calibration options, timelapse options, and AMS flags.

This is not a loose text protocol. This is a shared C++ ABI with shared data structures. The AGPL program defines the structures, and the closed plugin receives and acts on them.

This matters because AGPL v3 specifically refers to dynamic components that the work is designed to require through "intimate data communication or control flow".

---

## 8. File-transfer ABI is initialized from the same closed module

After loading `bambu_networking`, Bambu Studio calls:

```cpp
InitFTModule(networking_module);
```

This happens here:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/NetworkAgent.cpp#L280-L281

`FileTransferUtils.hpp` then defines another ABI around tunnel handles, job handles, job messages, JSON payloads, binary buffers, and file-transfer functions:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/FileTransferUtils.hpp#L25-L45

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/FileTransferUtils.hpp#L71-L90

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/src/slic3r/Utils/FileTransferUtils.hpp#L239-L252

This further confirms that the closed networking module is not only about login or cloud metadata. It is part of the runtime architecture used for file-transfer and print workflows.

---

## 9. Why this is an AGPL problem

Bambu Studio is licensed under AGPL v3. Bambu Lab's own README states that Bambu Studio is AGPL and that the `bambu networking plugin` is based on non-free libraries:

https://github.com/bambulab/BambuStudio/blob/e8c7dc1b84f5e3816238e070e04eeeb67cd92783/README.md#L42-L52

AGPL v3 defines "Corresponding Source" broadly. In section 1, it says that Corresponding Source includes interface definition files associated with the work and source code for shared libraries and dynamically linked subprograms that the work is specifically designed to require, for example through intimate data communication or control flow:

https://www.gnu.org/licenses/agpl-3.0.en.html#section1

That is exactly what Bambu Studio's public source shows:

- the AGPL program knows the exact closed library name `bambu_networking`,
- the AGPL program knows the agent name and version,
- the AGPL program dynamically loads the closed library into its own process,
- the AGPL program resolves `bambu_network_*` functions from that library,
- the AGPL program defines shared C++ structures used by that library,
- the AGPL program passes C++ callbacks to that library,
- the AGPL program gives the library a callback path into the main UI thread,
- the AGPL program initializes file-transfer ABI from the same module,
- the AGPL program uses the library in central workflows such as login, monitoring, LAN/cloud print, camera functions, device messaging, presets, filaments, telemetry, and file transfer.

This is not a "mere aggregate". This is not two independent programs sitting next to each other. This is a dynamic runtime component integrated into the AGPL application.

The FSF GPL FAQ describes the same principle for plugins: when plugins are dynamically linked, make function calls to each other, and share data structures, the FSF treats them as forming one combined program rather than a separate aggregate:

https://www.gnu.org/licenses/gpl-faq.html#GPLPlugins

Therefore, in my opinion, `bambu_networking` should be part of the Corresponding Source for Bambu Studio. If Bambu Studio is distributed as AGPL while `bambu_networking` is supplied only as closed binary code without complete source under AGPL-compatible terms, that is an AGPL compliance problem.

---

## 10. Why this matters for my defense

This is why Bambu Lab's accusation against me is backwards.

I worked on public AGPL code. I did not redistribute the closed plugin. I did not decompile it. I did not patch it. I did not need to inspect it.

Bambu Studio's own public AGPL source code shows the entire integration path: dynamic loading, symbol resolution, shared C++ structures, callbacks, main-thread callback, and file-transfer ABI.

If Bambu Lab says that my use of that path was not allowed, then Bambu Lab should first explain why its AGPL source code contains such a deep integration with a closed plugin, and why a user of AGPL code should not be allowed to work with the public AGPL side of that code.

Bambu Lab cannot have it both ways:

- If `bambu_networking` is separate and optional, then my AGPL fork should not be treated as responsible for the closed plugin's Terms of Use or closed runtime behavior, especially when my repository did not contain the plugin binaries.
- If `bambu_networking` is not separate and is actually an integrated runtime component of Bambu Studio, then Bambu Lab has an even bigger AGPL problem, because the plugin should be part of the Corresponding Source.

In both cases, a broad accusation against my AGPL source-code work is weak.

---

## 11. Bambu Lab's ToS claims need precision

I am not claiming that Bambu Lab cannot have Terms of Use for its private cloud service. A private cloud service and an open-source software license are separate topics.

However, Terms of Use cannot be used as a general ban on modifying or distributing AGPL source code. If Bambu Lab released Bambu Studio under AGPL, then recipients have rights under AGPL. AGPL grants rights to run, modify, and propagate the covered work, subject to the license terms. It also prohibits imposing further restrictions on the exercise of rights granted by the license.

If Bambu Lab claims that I violated its Terms of Use, it should clearly identify:

- which exact clause was allegedly violated,
- whether the allegation concerns AGPL source code, cloud service access, trademark, client identity, endpoint access, or something else,
- which exact technological protection measure was allegedly bypassed.

So far, I have not received a precise technical explanation. I received broad statements about Terms of Use, 17 U.S.C. Section 1201, "deceptive way", "acts like Bambu Studio", and "reverse engineered", but not a concrete analysis of my actual code.

---

## 12. EU law and interoperability

I am located in the European Union, so EU law is especially relevant.

Directive 2009/24/EC on the legal protection of computer programs is important here. It recognizes copyright protection for computer programs, but it also contains important limits and interoperability rules.

Article 1(2) states that ideas and principles underlying any element of a computer program, including those underlying its interfaces, are not protected by copyright under the Directive.

Article 5(3) allows a person having a right to use a copy of a program to observe, study, or test the functioning of the program in order to determine the ideas and principles underlying any element of the program, when doing acts they are entitled to do.

Article 6 provides a decompilation exception where it is indispensable to obtain information necessary to achieve interoperability of an independently created computer program with other programs, subject to the conditions in that article.

Article 8 states that contractual provisions contrary to Article 6 or to the exceptions in Article 5(2) and 5(3) are null and void.

Source:

https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32009L0024

This matters because even EU copyright law recognizes that interoperability and studying program behavior can be lawful in defined circumstances.

However, in my case the point is even simpler: I am not relying on a claim that I decompiled the plugin. I did not decompile it. I did not need to. The path I used was already present in Bambu Studio's public AGPL source code.

So the question is not "was I allowed to reverse-engineer the closed plugin?" The correct question is: why would I need to reverse-engineer it at all, when the relevant integration path was already published by Bambu Lab under AGPL?

---

## 13. 17 U.S.C. Section 1201

Bambu Lab also referred to 17 U.S.C. Section 1201.

Section 1201 concerns circumvention of a technological measure that effectively controls access to a copyrighted work:

https://www.law.cornell.edu/uscode/text/17/1201

I do not see where Bambu Lab has identified the exact effective technological measure that I allegedly bypassed. I also do not see where Bambu Lab has identified the copyrighted work whose access was allegedly controlled by that measure in this specific situation.

I did not patch a closed binary. I did not remove encryption. I did not redistribute the plugin. I did not base my work on decompilation of `bambu_networking`.

Also, during the Reddit conversation, Bambu Lab stated that the reason the path still worked was simply that they had not disabled it yet. That is important. If a path works because it has not been disabled, that is not the same as bypassing an effective, already-operating technical protection measure.

AGPL v3 also contains an anti-circumvention clause. Section 3 says that no covered work shall be deemed part of an effective technological measure under laws implementing anti-circumvention obligations, and that when conveying a covered work, the distributor waives legal power to forbid circumvention to the extent such circumvention is effected by exercising rights under the license:

https://www.gnu.org/licenses/agpl-3.0.en.html#section3

I am not claiming that this gives automatic access to Bambu Lab's private cloud. It does not. But it is relevant when Bambu Lab tries to describe work on AGPL-covered source code as anti-circumvention.

---

## 14. Cloud access and AGPL are separate issues

I am not claiming that AGPL automatically forces Bambu Lab to allow every fork into its private cloud.

Bambu Lab's cloud is a private service. Bambu Lab can define service rules for that service, subject to applicable law.

But that is separate from the AGPL source-code issue.

Bambu Lab itself publicly wrote that Bambu Studio is AGPL, that anyone can take its code, modify it, and distribute it, and that this is how the license and open source work:

https://blog.bambulab.com/setting-the-record-straight-on-cloud-access-and-community/

So if the allegation is about cloud access, Bambu Lab should frame it precisely as a cloud-service allegation. It should not present work on AGPL source code as if modifying Bambu Studio were itself improper.

And if the allegation depends on `bambu_networking`, then Bambu Lab should also answer the AGPL problem: why is a closed dynamic runtime component, deeply integrated with an AGPL application, being supplied without complete Corresponding Source under AGPL-compatible terms?

---

## 15. Authorization controls and the Linux path

Bambu Lab publicly described its authorization controls as applying to critical operations such as LAN/cloud print job initiation, remote video, firmware upgrade, motion controls, temperature controls, fan controls, AMS controls, and calibration controls. It also wrote that starting a print job using SD cards remains outside that authorization mechanism:

https://blog.bambulab.com/firmware-update-introducing-new-authorization-control-system-2/

Bambu Lab also wrote that third-party slicers based on open-source Studio development would no longer be able to use Studio's network plugin API for authorization control and that Bambu Connect would act as a replacement:

https://blog.bambulab.com/firmware-update-introducing-new-authorization-control-system-2/#Network-Plugin-for-Third-party-Slicer

That may be Bambu Lab's current product direction, but it does not change what I did.

I used an existing public AGPL code path. I did not add a new printer-side command class. I did not introduce a new hidden protocol. I did not bypass a security measure by breaking it. I used a path that existed and worked.

If Bambu Lab later wanted to disable that path, change the plugin, or move users to Bambu Connect, that is their product decision. It does not turn my work on public AGPL code into reverse engineering of the proprietary plugin.

---

## 16. Why I removed the repository

I removed the repository voluntarily, in good faith, and without admitting that Bambu Lab's allegations were correct.

I did it because I did not want to escalate the situation unnecessarily, especially after receiving broad legal threats and general references to Terms of Use and 17 U.S.C. Section 1201 without a precise technical explanation.

After removing the repository, I started analyzing Bambu Studio's public source code more carefully to understand the problem. That analysis led me to the conclusion that the central issue is not my work on AGPL source code. The central issue is Bambu Lab's own architecture: an AGPL application deeply integrated with a closed `bambu_networking` runtime component.

---

## 17. My conclusion

I reject the claim that I reverse-engineered the closed `bambu_networking` plugin.

I reject the claim that WSL2 / Lima are evidence of reverse engineering. They show the opposite: I used them to stay on the existing Linux-side AGPL path instead of touching the closed plugin.

I reject the claim that my work introduced new printer-side command classes or new unauthorized functionality beyond what followed from the existing Bambu Studio networking workflow.

I reject broad claims that Bambu Lab's Terms of Use can be used as a general prohibition on modifying AGPL source code.

My position is simple:

- I worked on public AGPL Bambu Studio source code.
- I did not redistribute closed `bambu_networking` binaries.
- I did not reverse-engineer `bambu_networking`.
- I did not patch or modify `bambu_networking`.
- I did not bypass an active protection mechanism.
- I used the existing Linux path because it was part of the AGPL code and it worked.

If Bambu Lab believes otherwise, it should identify the exact code, the exact Terms of Use clause, the exact technical protection measure, and the exact legal basis.

General statements about "reverse engineering", "deceptive way", "impersonation", and "bypass" are not enough.

At the same time, Bambu Studio's own public source shows that `bambu_networking` is deeply integrated with the AGPL application through dynamic loading, symbol resolution, shared C++ structures, callbacks, a main UI-thread callback, file-transfer ABI, and central networking workflows.

That is why I believe the real AGPL compliance problem is Bambu Lab's closed `bambu_networking` component, not my work on public AGPL source code.
