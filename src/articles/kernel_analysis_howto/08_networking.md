 Next Previous Contents
8. Linux Networking
8.1 How Linux networking is managed?

There exists a device driver for each kind of NIC. Inside it, Linux will ALWAYS call a standard high level routing: "netif_rx [net/core/dev.c]", which will controls what 3 level protocol the frame belong to, and it will call the right 3 level function (so we'll use a pointer to the function to determine which is right).

8.2 TCP example

We'll see now an example of what happens when we send a TCP packet to Linux, starting from ''netif_rx [net/core/dev.c]'' call.

Interrupt management: "netif_rx"

|netif_rx
   |__skb_queue_tail
      |qlen++
      |* simple pointer insertion *    
   |cpu_raise_softirq
      |softirq_active(cpu) |= (1 << NET_RX_SOFTIRQ) // set bit NET_RX_SOFTIRQ in the BH vector
 

Functions:

    __skb_queue_tail [include/linux/skbuff.h]
    cpu_raise_softirq [kernel/softirq.c]

Post Interrupt management: "net_rx_action"

Once IRQ interaction is ended, we need to follow the next part of the frame life and examine what NET_RX_SOFTIRQ does.

We will next call ''net_rx_action [net/core/dev.c]'' according to "net_dev_init [net/core/dev.c]".

|net_rx_action
   |skb = __skb_dequeue (the exact opposite of __skb_queue_tail)
   |for (ptype = first_protocol; ptype < max_protocol; ptype++) // Determine 
      |if (skb->protocol == ptype)                               // what is the network protocol
         |ptype->func -> ip_rcv // according to ''struct ip_packet_type [net/ipv4/ip_output.c]''
 
    **** NOW WE KNOW THAT PACKET IS IP ****
         |ip_rcv
            |NF_HOOK (ip_rcv_finish)
               |ip_route_input // search from routing table to determine function to call
                  |skb->dst->input -> ip_local_deliver // according to previous routing table check, destination is local machine
                     |ip_defrag // reassembles IP fragments
                        |NF_HOOK (ip_local_deliver_finish)
                           |ipprot->handler -> tcp_v4_rcv // according to ''tcp_protocol [include/net/protocol.c]''
 
     **** NOW WE KNOW THAT PACKET IS TCP ****
                           |tcp_v4_rcv   
                              |sk = __tcp_v4_lookup 
                              |tcp_v4_do_rcv
                                 |switch(sk->state) 

     *** Packet can be sent to the task which uses relative socket ***
                                 |case TCP_ESTABLISHED:
                                    |tcp_rcv_established
                                       |__skb_queue_tail // enqueue packet to socket
                                       |sk->data_ready -> sock_def_readable 
                                          |wake_up_interruptible
                                

     *** Packet has still to be handshaked by 3-way TCP handshake ***
                                 |case TCP_LISTEN:
                                    |tcp_v4_hnd_req
                                       |tcp_v4_search_req
                                       |tcp_check_req
                                          |syn_recv_sock -> tcp_v4_syn_recv_sock
                                       |__tcp_v4_lookup_established
                                 |tcp_rcv_state_process

                    *** 3-Way TCP Handshake ***
                                    |switch(sk->state)
                                    |case TCP_LISTEN: // We received SYN
                                       |conn_request -> tcp_v4_conn_request
                                          |tcp_v4_send_synack // Send SYN + ACK
                                             |tcp_v4_synq_add // set SYN state
                                    |case TCP_SYN_SENT: // we received SYN + ACK
                                       |tcp_rcv_synsent_state_process
                                          tcp_set_state(TCP_ESTABLISHED)
                                             |tcp_send_ack
                                                |tcp_transmit_skb
                                                   |queue_xmit -> ip_queue_xmit
                                                      |ip_queue_xmit2
                                                         |skb->dst->output
                                    |case TCP_SYN_RECV: // We received ACK
                                       |if (ACK)
                                          |tcp_set_state(TCP_ESTABLISHED)
                              

Functions can be found under:

    net_rx_action [net/core/dev.c]
    __skb_dequeue [include/linux/skbuff.h]
    ip_rcv [net/ipv4/ip_input.c]
    NF_HOOK -> nf_hook_slow [net/core/netfilter.c]
    ip_rcv_finish [net/ipv4/ip_input.c]
    ip_route_input [net/ipv4/route.c]
    ip_local_deliver [net/ipv4/ip_input.c]
    ip_defrag [net/ipv4/ip_fragment.c]
    ip_local_deliver_finish [net/ipv4/ip_input.c]
    tcp_v4_rcv [net/ipv4/tcp_ipv4.c]
    __tcp_v4_lookup
    tcp_v4_do_rcv
    tcp_rcv_established [net/ipv4/tcp_input.c]
    __skb_queue_tail [include/linux/skbuff.h]
    sock_def_readable [net/core/sock.c]
    wake_up_interruptible [include/linux/sched.h]
    tcp_v4_hnd_req [net/ipv4/tcp_ipv4.c]
    tcp_v4_search_req
    tcp_check_req
    tcp_v4_syn_recv_sock
    __tcp_v4_lookup_established
    tcp_rcv_state_process [net/ipv4/tcp_input.c]
    tcp_v4_conn_request [net/ipv4/tcp_ipv4.c]
    tcp_v4_send_synack
    tcp_v4_synq_add
    tcp_rcv_synsent_state_process [net/ipv4/tcp_input.c]
    tcp_set_state [include/net/tcp.h]
    tcp_send_ack [net/ipv4/tcp_output.c]

Description:

    First we determine protocol type (IP, then TCP)
    NF_HOOK (function) is a wrapper routine that first manages the network filter (for example firewall), then it calls ''function''.
    After we manage 3-way TCP Handshake which consists of:

SERVER (LISTENING)                       CLIENT (CONNECTING)
                           SYN 
                   <-------------------
 
 
                        SYN + ACK
                   ------------------->

 
                           ACK 
                   <-------------------

                    3-Way TCP handshake

    In the end we only have to launch "tcp_rcv_established [net/ipv4/tcp_input.c]" which gives the packet to the user socket and wakes it up.

Next Previous Contents 

