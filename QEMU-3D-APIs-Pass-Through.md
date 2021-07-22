## QEMU Virtual Machine APIs Pass-Through for 3D acceleration
Full device emulation is expensive in both implementation and performance, highly dependent on the quality and accuracy of available documentation of the device to be emulated. For GPUs in particular, they are scarce and well guarded secrets for most companies that designed them. Para-virtualized device emulation requires completely new sets of device drivers to be written for supported OSes, which works well for open source OSes such as Linux but unfeasible for closed source, unsupported legacy Windows OSes.

API "Pass-Through" or "Forwarding" has the advantages of performance, simplicity in implementation and maintenance. It works well for 3D acceleration APIs with user-space libraries, such as **3Dfx Glide** and **OpenGL**. Only simple kernel driver is needed for memory-mapping of the virtual interfaces for APIs data marshalling. The FIFO model virtual interfaces are capable of fully leveraging the performance characteristics of hardware acceleration for modern virtualization through various architecture extensions on modern CPUs. Today, all modern OSes provide CPU vendor agnostic user-space APIs for virtualization, **KVM on Linux**, **WHPX on Windows 10** and **HVF on Apple macOS**.
```
1. First Party Pass-Through

  Host OS
\-------------------------\
|    QEMU                 |
|  \-------------------\  |
|  |   Windows Guest   |  |   Note:
|  |                   |  |
|  | ( stubs )         |  |   MESA Gallium9 Pass-Through
|  | |- glide.dll      |  |   is an unimplemented, conceptual
|  | |- glide2x.dll    |  |   Direct3D9 API pass-through.
|  | |- glide3x.dll    |  |
|  | |- opengl32.dll   |  |
|  | \- d3d9.dll       |  |
|  |                   |  |
|  \--|----------------\  \---------------------------------\
|     V                                                     |
|   \--------------\  \--------------\  \---------------\   |
|   |    Glide     |--|   MESA GL    |--| MESA Gallium9 |   |
|   | Pass-Through |  | Pass-Through |  | Pass-Through  |   |
|   \---|----------\  \---|----------\  \---|-----------\   |
|       V                 V                 V               |
|     dgVoodoo2           Host            Gallium           |
|     nGlide              OpenGL          d3dadapter9       |
|     OpenGlide                                             |
|     psVoodoo                                              |
\-----------------------------------------------------------\


2. Third Party Pass-Through

  Host OS
\----------------------------------\
|    QEMU                          |
|  \----------------------------\  |
|  |   Windows Guest            |  |   Note:
|  |                            |  |
|  | + WineD3D                  |  |   MESA GL Pass-Through supports any
|  | + OpenGlide                |  |   Host OS with OpenGL desktop profiles
|  | + Zeckensack GlideWrapper  |  |   and has been qualified on
|  | + Svens Glide3-to-OpenGL   |  |
|  | |                          |  |    * Windows 10
|  | ( stubs )                  |  |    * Linux
|  | \- opengl32.dll            |  |    * macOS Big Sur
|  |                            |  |
|  \--|-------------------------\  |   across multiple GPU vendors with
|     V                            |   open-source or propriety GPU drivers
|   \--------------\               |
|   |   MESA GL    |               |    * Intel (i915, i965, iris)
|   | Pass-Through |               |    * NVIDIA (nouveau, nvidia)
|   \---|----------\               |    * AMD (r600, amdgpu)
|       V                          |    * Apple M1
|       Host                       |
|       OpenGL                     |
\----------------------------------\
```
