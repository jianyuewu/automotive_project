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
This command is particularly useful for developers to analyze and optimize the rendering performance of their apps. When you run adb shell dumpsys gfxinfo <package_name>,  
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

# Reference
https://time.geekbang.org/column/article/81049
