# VRTool-Reversed
private pasted vrchat client

Vrtool may appear to be a safe and private client, however, much of its code is stolen, and it is not truly undetected. Even though users may not experience an immediate ban, they are still putting themselves at risk of being detected by EAC that takes into account a variety of detection vectors.

This post will discuss the actions of the creator of VRTool and their lack of comprehension of the implications of their work. A brief overview of the reversal process will be provided, though I will not delve into an exhaustive analysis of the code, as I do not have the time to commit to such an endeavor.

VRTool uses a mapper called kdu, created by hfiref0x (https://github.com/hfiref0x/KDU), which utilizes exallocatepool. This makes it easy for EAC (Easy Anti-Cheat) to detect if the module is not being backed by a legitimate driver.

**Issues related to KDU:**
1. The kdu mapper utilizes a vulnerable driver that leaves traces after mapping, making it easy to detect by EAC.

2. Miincs driver is not backed by valid memory, which can also be detected by EAC.

3. Miincs driver creates a system thread that is also not backed by valid memory, making it detectable by EAC.

4. Miincs driver uses ObRegisterCallbacks from an invalid module, potentially even patching patchguard, which is known to have a "bypass" that EAC can detect and result in a ban. 

6. The system thread created by the driver is caught by EAC's NMI Callback, and since the driver is executing inside of invalid memory, it can cause a flag or ban by EAC.

7. The creator of VRtool appears to lack a comprehensive understanding of how to properly implement a driver mapper, as they unnecessarily extract the necessary DLLs for kdu from the loader and temporarily save them onto the disk in the same folder, map the driver, and then promptly delete the files, indicating a lack of understanding of how to properly paste.

8. The driver is protected by vmprotect, which has been flagged by EAC since 2019. This protection is located within invalid memory, and EAC has a signature specifically for vmprotected drivers, making it another detection vector that can lead to a future ban.

**Issues related to his dll mapping and injection method and client.**
1. The DLL mapping and injection has several issues, one of which is the allocation of 33MB of memory using RWX! provided by ZwAllocateVirtualMemory. This allocation is immediately detected by EAC when it maps this memory into the process, aswell as using KeAttachStackProcess in the process, resulting in two detections in one.

2. All the dependencies that have been loaded into the game, which are not necessary for the game to function but only for the client like "VTCore.dll" or "VTDictionary.dll", suddenly exist in the module list! An effective detection vector would be to inspect the list of loaded modules, as well as any suspiciously large RWX segments backed by non valid memory.

Heres a list of all the modules that were pulled from VRChat.exe that were in use by his client. These files will be provided in this repo.

![image](https://user-images.githubusercontent.com/45796777/213983724-64e14696-b84f-4c04-b04a-0e21ca7a2a11.png)


**The exports of VTLoader.dll**

His client appears to be calling an external function from a C++ Dynamic Link Library (DLL) that has been manually mapped. Upon further investigation, one can further assess what else is being called during the initialization process.
![image](https://user-images.githubusercontent.com/45796777/213985560-19c113c6-fa92-4974-8268-8da995c48c87.png)

**He is also utilizing Minhook to hook .txt, which creates a new detection vector from EAC for each hook.**
![image](https://user-images.githubusercontent.com/45796777/213988200-f168c079-71d3-4220-b259-96f11d1b4088.png)

You will suddenly see it is talking about hooking LoadLibrary, (Kernel32.dll) If the attempt to hook LoadLibrary fails, an error message will be displayed.

![image](https://user-images.githubusercontent.com/45796777/213986225-84269b4e-666a-4d61-9276-5ec521328f9f.png)

Once you are inside of his loadlibrary hook you can basically see how he initalizes his loader.
![image](https://user-images.githubusercontent.com/45796777/213986740-4d85e4c9-23b1-43f1-a68f-b3b709ff81f5.png)

Once again another massive detection vector as he goes in again for another .txt hook on Il2cppInit.
![image](https://user-images.githubusercontent.com/45796777/213986863-35db40ca-8458-4821-b009-239f5f04a45d.png)

Once we enter the il2cpp init hook, we can observe the standard initialization process of C# dlls.
![image](https://user-images.githubusercontent.com/45796777/213987406-59dcc03b-2bc4-4832-9912-2cb0444796ba.png)
