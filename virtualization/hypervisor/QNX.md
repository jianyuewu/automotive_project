# QNX hypervisor
Firstly by QSSL (QNX Software System Ltd.), now belongs to BlackBerry.  
QNX hypervisor is type-1 (bare metal).  
QNX is using micro kernel.  
# Main functionality
Mainly contains: QNX CAR Platform for Infotainment, QNX Platform for Digital Cockpits, QNX Platform for ADAS.  
QNX can support many guest functionalities, i.e. QNX OS for safety,  
    QNX OS, Linux, Android, Unmodified OS/ RTOS.  
Can provide Multi-layer security, InterVM Networking, Shared Memory, Adaptive Partitioning, Failure Handling/ Heartbeat, Shared Graphics/ Audio, Staggered boot.  
# Scheduling
QNS support 4 kinds of scheduling.  
1. FIFO  
2. Round Robin  
3. Adaptive Scheduling  
    If thread use up its time, still not blocked, then thread priority will -1, can only decrease once.  
    If thread becomes blocked, will come back to original priority.  
    So computing threads can have enough CPU resources, and fast response to user request.  
4. Sporadic Scheduling  
    Thread can run dynamically in normal priority or low priority. With parameters:  
    Initial Budget: Allowed exec time in normal priority.  
    Replenishment Period: Thread is allwed to use up its initial budget in this time period.  
    Max Number of Pending Replenishments: Restrict Replenishment Period happen times.  
    
