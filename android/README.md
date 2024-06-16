# UI
Compared to LCD screens, OLED screens have advantages in color, bendability, thickness and power consumption. Because of these advantages, full screens, curved screens and future flexible folding screens all use OLED materials.  
px->dpi  

Rendering:  
Similar to the painting process.  
Pen: Skia/OpenGL  
Paper: Surface  
Board: Graphic buffer, Triple buffering.  
Display: SurfaceFlinger.  

## Project Butter (Android 4.1)
VSYNC+Triple Buffering  

Triple Buffering:  
CPU buffer + GPU buffer + Display(3nd Graphic Buffer)  
=> Frame Buffer  

Traceview + systrace + Tracer for OpenGL ES  

## RenderThread (Android 5.0)
RenderNode + RenderThread  

## gfxinfo + hwui + Vulkun (Newer Android)
gfxinfo is a command used with adb shell (Android Debug Bridge) to gather information about the graphics performance of an application.  
This command is particularly useful for developers to analyze and optimize the rendering performance of their apps. When you run adb shell dumpsys gfxinfo pkg_name,  
it provides detailed output about the rendering time for each frame, helping to identify bottlenecks or inefficiencies.  

HWUI stands for Hardware Accelerated User Interface, which refers to the hardware acceleration support built into Android's drawing framework.  

Vulkan is a modern cross-platform graphics and compute API, developed by the Khronos Group.  
It is known for providing high-efficiency, low-overhead access to modern GPUs used in a wide variety of devices from PCs and consoles to mobile devices and embedded platforms.  
Vulkan gives developers more control over GPU operations and is designed to enable better performance and more balanced CPU/GPU usage.

## Rendering
60fps: 1000 ms／60 fps = 16 ms (rendering time, including UI thread + Render thread + Graphics).  
Test tools: Profile GPU Rendering + Show GPU Overdraw  
Debugger: systrace + Tracer for OpenGL ES + Graphics API Debugger (GAPID)  

```bash
adb shell dumpsys gfxinfo pkgname  
adb shell dumpsys gfxinfo pkgname framestats  
adb shell dumpsys SurfaceFlinger  
```

### optimization
HW offload:  
    SVG->Bitmap  
Create View:  
    1. Code/X2C  
    2. Async: Looper Msg Q -> UI Looper MsgQueue   
    3. View reuse  

# Native Hooks  
Hooks are modified at runtime, mainly find the library, and find the addr, then replace it.  
1. GOT/PLT Hook:  
PLT (Procedure Linkage Table): In code section, external func will have a record in PLT.  
GOT (Global Offset Table): Stores func offset, after relocate, will have absolute func addr.  
Facebook has open-sourced it's plt hook, and iqiyi open-sourced it's got hook.  

2. Trap hook:  
Ptrace, When the debugger is bound to the target program, any signal of the target program will be intercepted by the debugger first, and the debugger will have the opportunity to process the relevant signal, and then hand over the execution permission to the target program to continue execution. Facebook's Profilo is using SIGPROF signal to trace latency.  

```bash
readelf -d /usr/lib/libmultipath.so | grep NEEDED  
 0x0000000000000001 (NEEDED)             Shared library: [libdevmapper.so.1.02.1]  
 0x0000000000000001 (NEEDED)             Shared library: [libudev.so.1]  
 0x0000000000000001 (NEEDED)             Shared library: [libmpathcmd.so.0]  
 0x0000000000000001 (NEEDED)             Shared library: [liburcu.so.8]  
 0x0000000000000001 (NEEDED)             Shared library: [libaio.so.1]  
 0x0000000000000001 (NEEDED)             Shared library: [libsystemd.so.0]  
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]  
```

3. Inline hook  
Inline hook will update func beginning prologue with jump instruction, and jump to target hook func, and retain original function, and complete subsequent.
ByteDance has open-sourced its android inline hook.
It is not restricted by GOT/PLT table, so can hook any func in theory. But implementation is complicated, so not widely used.  

# Framework  
Components: Activity, Service, Broadcast, Contprovider.
Handler:
Binder:
Rendering:
Sharepreference:
View/ViewGroup:
Touch:
Services: AMS, PMS, WMS, 

# Reference
https://time.geekbang.org/column/article/81049  
https://time.geekbang.org/column/intro/100021101?tab=catalog  
https://github.com/bytedance/android-inline-hook  
https://github.com/bytedance/bhook  
https://github.com/iqiyi/xHook  
https://github.com/facebookincubator/profilo/tree/master/deps/plthooks  
https://github.com/facebookarchive/profilo  
https://eli.thegreenplace.net/2011/01/23/how-debuggers-work-part-1  
https://eli.thegreenplace.net/2011/01/27/how-debuggers-work-part-2-breakpoints  
https://eli.thegreenplace.net/2011/02/07/how-debuggers-work-part-3-debugging-information  
https://gcc.gnu.org/git/gitweb.cgi?p=gcc.git;a=blob;f=libgcc/unwind.inc;h=12f62bca7335f3738fb723f00b1175493ef46345;hb=HEAD#l275  