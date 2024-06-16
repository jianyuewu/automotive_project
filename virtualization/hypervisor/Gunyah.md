# Gunyah hypervisor
"Gunyah" is a language of Australian Aboriginal origin. The Gunyah hypervisor provides a reference configuration that can be used to create multiple trusted and dependent virtual machines (VMs). As a Type-1 hypervirtualizer, it operates independently of any high-level operating system kernel and operates at a higher level of CPU privilege than VMs, ensuring better security and a smaller trusted computing base.  

# Main functionality
Threading and scheduling: The scheduler is responsible for scheduling virtual CPUs on physical CPUs and implementing time sharing.  
Memory management: Track the ownership and usage of all controlled memory, and memory partitioning is the foundation of security between VMs.  
Interrupt Virtualization: All interrupts are handled in the hyper-virtualizer and routed to the specified VM.  
Inter-VM communication: Provides a variety of inter-VM communication mechanisms.  
Device virtualization: Supports para-virtualization of devices, using inter-VM communication, and emulation for some underlying system features and devices, such as interrupt controllers.  
