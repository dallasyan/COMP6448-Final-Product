---
title: Creating Your Own Logs
linktitle: Creating Your Own Logs
type: book
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

This section is about creating logs regarding network activity to and from a machine. It is intended for educational purposes for those who are interested in SOC operations and log analysis but do not have access to heavily customisable log collection tools. These instructions are not suitable for production.

## eBPF

Berkeley Packet Filter (BPF) is essentially a Linux technology to filter and mangle packets through kernel modules. You can read more about it on [the Wikipedia page](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter). Often it is used for fast packet filtering as it is very low level (for example, packets will reach BPF programs before [uncomplicated firewall](https://wiki.ubuntu.com/UncomplicatedFirewall)), tunneling (e.g. IP in IP encapsulation and decapsulation) or other packet manipulation such as network address translation.

BPF kernel modules are able to communicate with the user space through maps. We can use maps to store custom information about packets, which a program in the user space can read and format into logs.

There are many resources that document how to write and load BPF programs, I will be focusing on how they can be used to collect information about network activity so I will link some resources here.

[Here](https://liuhangbin.netlify.app/post/ebpf-and-xdp/) is a particularly good tutorial on writing BPF programs, and also has some instructions on how to compile out of kernel.\
[Here](https://blogs.oracle.com/linux/notes-on-bpf-1) is part one of a seven part series on how to write BPF programs.\
[Here](https://github.com/shemminger/iproute2/tree/main/examples/bpf) are the BPF examples from iproute2. In particular, the map in map example may be useful.\
[This](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md) details the Linux kernel versions required for many important BPF functions.

The Linux repository has some BPF examples [here](https://github.com/torvalds/linux/tree/master/samples/bpf) but they use a different syntax to what we are interested in. We want to use iproute2 to load our kernel modules.

## Collecting Information

### Kernel Space

Using an eBPF program loaded with tc, we can collect the source IPs of any IPv4 TCP packets, while simultaneously blocking all other packets.

```c
#include <linux/in.h>
#include <linux/if_ether.h>
#include <linux/ip.h>
#include <linux/tcp.h>
#include <linux/udp.h>
#include "bpf_api.h" // This is the BPF interface from iproutes2, I find it easier to use than other interfaces

#ifndef __inline
# define __inline                   \
    inline __attribute__((always_inline))
#endif

// Declare the map
struct bpf_elf_map __section_maps ips = {
    .type         = BPF_MAP_TYPE_HASH, // This is a hashmap (as opposed to an array). Using an array instead may require things such as spinlocks to solve the producer/consumer problem.
    .size_key     = sizeof(uint32_t), // The key is an IP address, which is an unsigned integer. You can use structs as well so you can store IP/port or something similar too.
    .size_value   = sizeof(char), // We are basically using this map as a set, so the value doesn't matter to us
    .pinning      = PIN_GLOBAL_NS, // Use the global namespace. Pinning is required for user space programs to be able to access the map
    .max_elem     = 65565, // Maximum number of elements in the map at once.
};

__section("ingress")
int tc_ingress(struct __sk_buff *skb) {
    // Pulls all the data into linear memory
    skb_pull_data(skb, 0);

    // Only allow IPv4, TC_ACT_STOLEN basically drops the packet
    if (skb->protocol != __constant_htons(ETH_P_IP)) {
        return TC_ACT_STOLEN;
    }

    void *data_end = (void *)(long)skb->data_end;
    void *data = (void *)(long)skb->data;

    // Get the IPv4 header
    struct iphdr *iph = data + sizeof(struct ethhdr);

    // These kinds of checks (of type x > data_end) are required whenever accessing skb data.
    // As eBPF programs are loaded into kernel, there are strict requirements such that you don't access memory out of bounds.
    // These requirements are checked through some kind of state checking before the module is loaded.
    if ((void *)(iph + 1) > data_end) {
        return TC_ACT_STOLEN;
    }

    // Source IP
    uint32_t src_ip = iph->saddr;

    // Protocol
    uint8_t proto = iph->protocol;

    // Check the protocol is TCP, then try to add it to the map
    // If map_update_elem is non-zero, that means it failed (e.g. map is full)
    if (proto == IPPROTO_TCP) {
        if (map_update_elem(&ips, &src_ip, &tem, BPF_ANY)) {
            // printt writes to the pipe at /sys/kernel/debug/tracing/trace_pipe
            // sudo cat /sys/kernel/debug/tracing/trace_pipe to view messages
            printt("ERROR ADDING IP %x\n", src_ip);
        }
        return TC_ACT_OK;
    }

    return TC_ACT_STOLEN;
}

BPF_LICENSE("GPL");
```

Note that this module only has an ingress section, so will only affect incoming packets. The kernel module can be customised to only collect data from IPs that try to access a specific port, or from certain IP ranges.

### User Space

In the user space we can write a program that takes all the IPs from the kernel module and prints them out.

```c
#include <stdio.h>
#include <string.h>
#include <assert.h>
#include <unistd.h>
#include <stdlib.h>
#include <uapi/linux/bpf.h>
#include <stdint.h>
#include <asm/byteorder.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/ioctl.h>
#include <netinet/in.h>
#include <net/if.h>
#include <arpa/inet.h>
#include <time.h>

#include "libbpf.h"

// Default map location
#define BPF_MAP_LOC "/sys/fs/bpf/tc/globals/"

enum bpf_map_num {
    ips
};

int main() {
    char *bpf_maps[] = {"ips"};

    char buffer[1024] = BPF_MAP_LOC;
    int fd[1] = {0};

    // Load maps
    for (int i = 0; i < 1; i++) {
        strcpy(buffer + strlen(BPF_MAP_LOC), bpf_maps[i]);
        fd[i] = bpf_obj_get(buffer);
    }

    while (1) {
        uint32_t key = {0};
        uint32_t next_key;
        // bpf_map_get_next_key iterates through all keys of the map
        // If the pointer you give has dereferenced value 0 in the second argument, it takes the first key
        // Otherwise it takes the next key after the one in the second argument
        // Result is stored in the third argument
        // Returns 0 on success
        while (!bpf_map_get_next_key(fd[ips], &key, &next_key)) {
            key = next_key;
            printf("%u\n", key); // Print the IP. Note that it will be in network endianness, so will require ntohl to be read correctly.
            bpf_map_delete_elem(fd[ips], &key); // Delete the IP
        }
    }
    return 0;
}
```

Instead of printing the IP we should add it to a log file or a database.

### Timestamp

You can also customise the data collected. For example, the BPF helper function `bpf_ktime_get_ns()` retrieves the time in nanoseconds since bootup. In the user space you can get the nanoseconds since epoch that the bootup time is by subtracting `clock_gettime(CLOCK_MONOTONIC,)` from `clock_gettime(CLOCK_REALTIME,)`. So you can pass the time since bootup into the map to the user space, which can add on to get time since epoch and add a timestamp to the log.

### Packet Mangling

BPF can also be used to mangle packets. For example if you detect a suspicious packet you can change its destination IP and/or port. The following function changes the destination IP and port of a TCP packet.

```c
#define IP_CSUM_OFF (ETH_HLEN + offsetof(struct iphdr, check))
#define IP_DEST_OFF (ETH_HLEN + offsetof(struct iphdr, daddr))
#define TCP_CSUM_OFF (ETH_HLEN + sizeof(struct iphdr) + offsetof(struct tcphdr, check))
#define TCP_DPORT_OFF (ETH_HLEN + sizeof(struct iphdr) + offsetof(struct tcphdr, dest))

static __always_inline void set_destination_ip_and_port(struct __sk_buff *skb, uint32_t new_ip, uint16_t new_port) {
    uint32_t ip_loc = IP_DEST_OFF;
    uint32_t tcp_loc = TCP_DPORT_OFF;
    uint32_t tcp_check_loc = TCP_CSUM_OFF;
    uint32_t old_ip = __constant_htonl(load_word(skb, ip_loc));

    // TCP checksum for changing IP
    l4_csum_replace(skb, tcp_check_loc, old_ip, new_ip, BPF_F_PSEUDO_HDR | sizeof(new_ip));

    // IPv4 checksum
    l3_csum_replace(skb, IP_CSUM_OFF, old_ip, new_ip, sizeof(new_ip));

    // Set the IP
    skb_store_bytes(skb, ip_loc, &new_ip, sizeof(new_ip), 0);

    uint16_t old_port = __constant_htons(load_half(skb, tcp_loc));

    // TCP checksum for changing port
    l4_csum_replace(skb, tcp_check_loc, old_port, new_port, sizeof(new_port));

    // Set the port
    skb_store_bytes(skb, tcp_loc, &new_port, sizeof(new_port), 0);
}
```

## Creating Effective Logs

You can use multiple maps to create different logs, or use map in map to create a log file per IP address. As BPF can also act as a packet filter, you can also see which attacks you prevented.

Look at the [Security Correlation](/docs/security-correlation) page to get some ideas on the types of logs you can write. The BPF program should be able to handle firewall type logs, and you can write user space programs (in whatever language you wish) to handle IDS alerts and web server type logs if you want to.

For example, you can write a program that raises an alarm if there is a heavy amount of traffic from one IP.

It may also be useful to write a program to make performing manual searches through logs easier, as that is what you will be doing in the analysis phase.