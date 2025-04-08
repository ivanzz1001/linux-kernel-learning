# sk_buff数据结构解析

[sk_buff](https://elixir.bootlin.com/linux/v6.6.8/source/include/linux/skbuff.h#L728)是Linux网络中一个十分重要的数据结构，每一个数据包的发送与接收都要使用此数据结构来进行处理。

参看:

- [sk_buff 简介](https://www.llcblog.cn/2020/10/26/how-sk-buff-work/)

- [网络通信简介——sk_buff结构体介绍](https://www.cnblogs.com/theseventhson/p/15858194.html)

- [关键数据结构sk_buff](https://github.com/0voice/linux_kernel_wiki/tree/main/%E6%96%87%E7%AB%A0/%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE%E6%A0%88)

- [struct sk_buff](https://docs.kernel.org/networking/skbuff.html)

- [Linux内核网络源码解析: sk_buff结构](https://liu-jianhao.github.io/2019/05/linux%E5%86%85%E6%A0%B8%E7%BD%91%E7%BB%9C%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%901sk_buff%E7%BB%93%E6%9E%84/)

- [sk buffer](https://docs.kernel.org/networking/skbuff.html)

- [sk buffer source(Linux v6.6.8)](https://elixir.bootlin.com/linux/v6.6.8/source/include/linux/skbuff.h#L728)

- [linux内核TCP/IP源码浅析](https://blog.csdn.net/weixin_40355471/article/details/131535653)

为使在阅读过程中对`sk_buff`能有一个全局的视角，这里我们先贴出该结构的完整定义，然后再逐步来进行分析:

```C
/**
 *	struct sk_buff - socket buffer
 *	@next: Next buffer in list
 *	@prev: Previous buffer in list
 *	@tstamp: Time we arrived/left
 *	@skb_mstamp_ns: (aka @tstamp) earliest departure time; start point
 *		for retransmit timer
 *	@rbnode: RB tree node, alternative to next/prev for netem/tcp
 *	@list: queue head
 *	@ll_node: anchor in an llist (eg socket defer_list)
 *	@sk: Socket we are owned by
 *	@ip_defrag_offset: (aka @sk) alternate use of @sk, used in
 *		fragmentation management
 *	@dev: Device we arrived on/are leaving by
 *	@dev_scratch: (aka @dev) alternate use of @dev when @dev would be %NULL
 *	@cb: Control buffer. Free for use by every layer. Put private vars here
 *	@_skb_refdst: destination entry (with norefcount bit)
 *	@sp: the security path, used for xfrm
 *	@len: Length of actual data
 *	@data_len: Data length
 *	@mac_len: Length of link layer header
 *	@hdr_len: writable header length of cloned skb
 *	@csum: Checksum (must include start/offset pair)
 *	@csum_start: Offset from skb->head where checksumming should start
 *	@csum_offset: Offset from csum_start where checksum should be stored
 *	@priority: Packet queueing priority
 *	@ignore_df: allow local fragmentation
 *	@cloned: Head may be cloned (check refcnt to be sure)
 *	@ip_summed: Driver fed us an IP checksum
 *	@nohdr: Payload reference only, must not modify header
 *	@pkt_type: Packet class
 *	@fclone: skbuff clone status
 *	@ipvs_property: skbuff is owned by ipvs
 *	@inner_protocol_type: whether the inner protocol is
 *		ENCAP_TYPE_ETHER or ENCAP_TYPE_IPPROTO
 *	@remcsum_offload: remote checksum offload is enabled
 *	@offload_fwd_mark: Packet was L2-forwarded in hardware
 *	@offload_l3_fwd_mark: Packet was L3-forwarded in hardware
 *	@tc_skip_classify: do not classify packet. set by IFB device
 *	@tc_at_ingress: used within tc_classify to distinguish in/egress
 *	@redirected: packet was redirected by packet classifier
 *	@from_ingress: packet was redirected from the ingress path
 *	@nf_skip_egress: packet shall skip nf egress - see netfilter_netdev.h
 *	@peeked: this packet has been seen already, so stats have been
 *		done for it, don't do them again
 *	@nf_trace: netfilter packet trace flag
 *	@protocol: Packet protocol from driver
 *	@destructor: Destruct function
 *	@tcp_tsorted_anchor: list structure for TCP (tp->tsorted_sent_queue)
 *	@_sk_redir: socket redirection information for skmsg
 *	@_nfct: Associated connection, if any (with nfctinfo bits)
 *	@nf_bridge: Saved data about a bridged frame - see br_netfilter.c
 *	@skb_iif: ifindex of device we arrived on
 *	@tc_index: Traffic control index
 *	@hash: the packet hash
 *	@queue_mapping: Queue mapping for multiqueue devices
 *	@head_frag: skb was allocated from page fragments,
 *		not allocated by kmalloc() or vmalloc().
 *	@pfmemalloc: skbuff was allocated from PFMEMALLOC reserves
 *	@pp_recycle: mark the packet for recycling instead of freeing (implies
 *		page_pool support on driver)
 *	@active_extensions: active extensions (skb_ext_id types)
 *	@ndisc_nodetype: router type (from link layer)
 *	@ooo_okay: allow the mapping of a socket to a queue to be changed
 *	@l4_hash: indicate hash is a canonical 4-tuple hash over transport
 *		ports.
 *	@sw_hash: indicates hash was computed in software stack
 *	@wifi_acked_valid: wifi_acked was set
 *	@wifi_acked: whether frame was acked on wifi or not
 *	@no_fcs:  Request NIC to treat last 4 bytes as Ethernet FCS
 *	@encapsulation: indicates the inner headers in the skbuff are valid
 *	@encap_hdr_csum: software checksum is needed
 *	@csum_valid: checksum is already valid
 *	@csum_not_inet: use CRC32c to resolve CHECKSUM_PARTIAL
 *	@csum_complete_sw: checksum was completed by software
 *	@csum_level: indicates the number of consecutive checksums found in
 *		the packet minus one that have been verified as
 *		CHECKSUM_UNNECESSARY (max 3)
 *	@dst_pending_confirm: need to confirm neighbour
 *	@decrypted: Decrypted SKB
 *	@slow_gro: state present at GRO time, slower prepare step required
 *	@mono_delivery_time: When set, skb->tstamp has the
 *		delivery_time in mono clock base (i.e. EDT).  Otherwise, the
 *		skb->tstamp has the (rcv) timestamp at ingress and
 *		delivery_time at egress.
 *	@napi_id: id of the NAPI struct this skb came from
 *	@sender_cpu: (aka @napi_id) source CPU in XPS
 *	@alloc_cpu: CPU which did the skb allocation.
 *	@secmark: security marking
 *	@mark: Generic packet mark
 *	@reserved_tailroom: (aka @mark) number of bytes of free space available
 *		at the tail of an sk_buff
 *	@vlan_all: vlan fields (proto & tci)
 *	@vlan_proto: vlan encapsulation protocol
 *	@vlan_tci: vlan tag control information
 *	@inner_protocol: Protocol (encapsulation)
 *	@inner_ipproto: (aka @inner_protocol) stores ipproto when
 *		skb->inner_protocol_type == ENCAP_TYPE_IPPROTO;
 *	@inner_transport_header: Inner transport layer header (encapsulation)
 *	@inner_network_header: Network layer header (encapsulation)
 *	@inner_mac_header: Link layer header (encapsulation)
 *	@transport_header: Transport layer header
 *	@network_header: Network layer header
 *	@mac_header: Link layer header
 *	@kcov_handle: KCOV remote handle for remote coverage collection
 *	@tail: Tail pointer
 *	@end: End pointer
 *	@head: Head of buffer
 *	@data: Data head pointer
 *	@truesize: Buffer size
 *	@users: User count - see {datagram,tcp}.c
 *	@extensions: allocated extensions, valid if active_extensions is nonzero
 */

struct sk_buff {
	union {
		struct {
			/* These two members must be first to match sk_buff_head. */
			struct sk_buff		*next;
			struct sk_buff		*prev;

			union {
				struct net_device	*dev;
				/* Some protocols might use this space to store information,
				 * while device pointer would be NULL.
				 * UDP receive path is one user.
				 */
				unsigned long		dev_scratch;
			};
		};
		struct rb_node		rbnode; /* used in netem, ip4 defrag, and tcp stack */
		struct list_head	list;
		struct llist_node	ll_node;
	};

	union {
		struct sock		*sk;
		int			ip_defrag_offset;
	};

	union {
		ktime_t		tstamp;
		u64		skb_mstamp_ns; /* earliest departure time */
	};
	/*
	 * This is the control buffer. It is free to use for every
	 * layer. Please put your private variables there. If you
	 * want to keep them across layers you have to do a skb_clone()
	 * first. This is owned by whoever has the skb queued ATM.
	 */
	char			cb[48] __aligned(8);

	union {
		struct {
			unsigned long	_skb_refdst;
			void		(*destructor)(struct sk_buff *skb);
		};
		struct list_head	tcp_tsorted_anchor;
#ifdef CONFIG_NET_SOCK_MSG
		unsigned long		_sk_redir;
#endif
	};

#if defined(CONFIG_NF_CONNTRACK) || defined(CONFIG_NF_CONNTRACK_MODULE)
	unsigned long		 _nfct;
#endif
	unsigned int		len,
				data_len;
	__u16			mac_len,
				hdr_len;

	/* Following fields are _not_ copied in __copy_skb_header()
	 * Note that queue_mapping is here mostly to fill a hole.
	 */
	__u16			queue_mapping;

/* if you move cloned around you also must adapt those constants */
#ifdef __BIG_ENDIAN_BITFIELD
#define CLONED_MASK	(1 << 7)
#else
#define CLONED_MASK	1
#endif
#define CLONED_OFFSET		offsetof(struct sk_buff, __cloned_offset)

	/* private: */
	__u8			__cloned_offset[0];
	/* public: */
	__u8			cloned:1,
				nohdr:1,
				fclone:2,
				peeked:1,
				head_frag:1,
				pfmemalloc:1,
				pp_recycle:1; /* page_pool recycle indicator */
#ifdef CONFIG_SKB_EXTENSIONS
	__u8			active_extensions;
#endif

	/* Fields enclosed in headers group are copied
	 * using a single memcpy() in __copy_skb_header()
	 */
	struct_group(headers,

	/* private: */
	__u8			__pkt_type_offset[0];
	/* public: */
	__u8			pkt_type:3; /* see PKT_TYPE_MAX */
	__u8			ignore_df:1;
	__u8			dst_pending_confirm:1;
	__u8			ip_summed:2;
	__u8			ooo_okay:1;

	/* private: */
	__u8			__mono_tc_offset[0];
	/* public: */
	__u8			mono_delivery_time:1;	/* See SKB_MONO_DELIVERY_TIME_MASK */
#ifdef CONFIG_NET_XGRESS
	__u8			tc_at_ingress:1;	/* See TC_AT_INGRESS_MASK */
	__u8			tc_skip_classify:1;
#endif
	__u8			remcsum_offload:1;
	__u8			csum_complete_sw:1;
	__u8			csum_level:2;
	__u8			inner_protocol_type:1;

	__u8			l4_hash:1;
	__u8			sw_hash:1;
#ifdef CONFIG_WIRELESS
	__u8			wifi_acked_valid:1;
	__u8			wifi_acked:1;
#endif
	__u8			no_fcs:1;
	/* Indicates the inner headers are valid in the skbuff. */
	__u8			encapsulation:1;
	__u8			encap_hdr_csum:1;
	__u8			csum_valid:1;
#ifdef CONFIG_IPV6_NDISC_NODETYPE
	__u8			ndisc_nodetype:2;
#endif

#if IS_ENABLED(CONFIG_IP_VS)
	__u8			ipvs_property:1;
#endif
#if IS_ENABLED(CONFIG_NETFILTER_XT_TARGET_TRACE) || IS_ENABLED(CONFIG_NF_TABLES)
	__u8			nf_trace:1;
#endif
#ifdef CONFIG_NET_SWITCHDEV
	__u8			offload_fwd_mark:1;
	__u8			offload_l3_fwd_mark:1;
#endif
	__u8			redirected:1;
#ifdef CONFIG_NET_REDIRECT
	__u8			from_ingress:1;
#endif
#ifdef CONFIG_NETFILTER_SKIP_EGRESS
	__u8			nf_skip_egress:1;
#endif
#ifdef CONFIG_TLS_DEVICE
	__u8			decrypted:1;
#endif
	__u8			slow_gro:1;
#if IS_ENABLED(CONFIG_IP_SCTP)
	__u8			csum_not_inet:1;
#endif

#if defined(CONFIG_NET_SCHED) || defined(CONFIG_NET_XGRESS)
	__u16			tc_index;	/* traffic control index */
#endif

	u16			alloc_cpu;

	union {
		__wsum		csum;
		struct {
			__u16	csum_start;
			__u16	csum_offset;
		};
	};
	__u32			priority;
	int			skb_iif;
	__u32			hash;
	union {
		u32		vlan_all;
		struct {
			__be16	vlan_proto;
			__u16	vlan_tci;
		};
	};
#if defined(CONFIG_NET_RX_BUSY_POLL) || defined(CONFIG_XPS)
	union {
		unsigned int	napi_id;
		unsigned int	sender_cpu;
	};
#endif
#ifdef CONFIG_NETWORK_SECMARK
	__u32		secmark;
#endif

	union {
		__u32		mark;
		__u32		reserved_tailroom;
	};

	union {
		__be16		inner_protocol;
		__u8		inner_ipproto;
	};

	__u16			inner_transport_header;
	__u16			inner_network_header;
	__u16			inner_mac_header;

	__be16			protocol;
	__u16			transport_header;
	__u16			network_header;
	__u16			mac_header;

#ifdef CONFIG_KCOV
	u64			kcov_handle;
#endif

	); /* end headers group */

	/* These elements must be at the end, see alloc_skb() for details.  */
	sk_buff_data_t		tail;
	sk_buff_data_t		end;
	unsigned char		*head,
				*data;
	unsigned int		truesize;
	refcount_t		users;

#ifdef CONFIG_SKB_EXTENSIONS
	/* only useable after checking ->active_extensions != 0 */
	struct skb_ext		*extensions;
#endif
};
```

