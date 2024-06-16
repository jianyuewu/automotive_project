# Automotive OS basics
Automotive operating system mainly can be divided into 2 categories:  

1. Vehicle control operating system: Mainly realizes vehicle chassis control, power system and automatic driving;  
2. Intelligent cockpit operating system: Mainly provides a control platform for in-vehicle infotainment services and in-vehicle human-computer interaction, and is the operating environment for automobiles to realize the integration of cockpit intelligence and multi-source information.  

Those OS need suit for realtime, RAS(Reliability, Accessibility, Serviceability), Safety(IEC 61508, ISO26262).  
AUTOSAR(Automotive Open System Architecture) standard is widely used in those OS.  
Now mainly use QNX(Especially in safety related), which is expensive. VxWorks is also used in some company.  
There are also open source and free OS widely used in automotive. Like Android(some customed OS, i.e. Google Android Automotive OS, Huawei Harmony OS, Alibaba AliOS). Linux(i.e. Tesla Version OS, Volkswagen vw.OS) and AGL(Automotive Grade Linux).  

# AUTOSAR architecture
Consists of CP (Classic Platform) and AP (Adaptive Platform).  
Classic Platform:  
  Microcontroller, BSW (Basic Software Layer), ECUAL (ECU Abstraction Layer), Services Layer, CDD (Complex Device Drivers), RTE (Runtime Environment), ASW (Application Software layer).  
Adaptive Platform:  
  High performance Hardware/ Virtual Machine, ARA (AUTOSAR Runtime for Adaptive Application), Adaptive Application.  

# Hypervisor standards
BlackBerry: QNX hypervisor, Qualcomm Gunyah, Intel & Linux fundation ACRN, Virtio.  
Hypervisor is between DCU HW and OS SW.  
Now automotive is mainly using DCU (Domain Control Unit) instead of ECU, because ECU num is increasing, so bus will be longer. To reduce the CAN/LIN communication, more powerful DCU is used.  
ECU contains:  
EMS(Engine Management System), TCU(Telematics Control Unit), ESP(Electronic Stability Program).  

