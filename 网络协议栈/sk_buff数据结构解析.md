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

- [核心数据结构sk_buff](https://void-star.icu/archives/939)

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

# 1. sk_buff整体拓扑

sk_buff本身是一个元数据结构，其并不包含任何packet data。所有的数据都存放在所关联的buffer中。

![skbuff-top](https://raw.githubusercontent.com/ivanzz1001/linux-kernel-learning/master/%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE%E6%A0%88/image/sk_buff_1.png)

sk_buff由一个线性buffer(linear data buffer)和可选的page buffer所组成：

1. **线性Buffer**

    sk_buff.head到sk_buff.end之间的这段缓存称为线性Buffer，其由`head`、`data`、`tail`、`end`四个字段将这一部分空间分成三段:

      - headroom

      - data

      - tailroom

1. **Page Buffer**

    由skb_shared_info指向的这段空间称为page buffer。


# 2. 关键字段解析


## 2.1 链表与队列管理

```C
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
```

1. **next和prev**

    - 用途：构成双向链表，用于将多个sk_buff组织成队列（如接收队列`input_pkt_queue` 或发送队列`qdisc_priv`）。

    - 操作：通过 skb_queue_head/skb_queue_tail 等函数管理队列

    - 关键点：与 sk_buff_head 结构共享前两个字段，确保队列操作的高效性

1. **list和rbnode**

    - list：通用链表头，用于协议无关的链表管理（如软中断处理队列）。

    - rbnode：红黑树节点，用于需要高效查找的场景（如 TCP 重传队列、网络仿真 netem）。

1. **dev**

    指向接收或发送此数据包的网络设备（struct net_device）

1. **dev_scratch**

    当`dev`字段为NULL时，某些协议会使用此字段来存储一些信息，例如UDP协议:

    ```C
    /* UDP uses skb->dev_scratch to cache as much information as possible and avoid
     * possibly multiple cache miss on dequeue()
     */
    struct udp_dev_scratch {
	    /* skb->truesize and the stateless bit are embedded in a single field;
	     * do not use a bitfield since the compiler emits better/smaller code
	     * this way
	     */
	    u32 _tsize_state;

    #if BITS_PER_LONG == 64
	    /* len and the bit needed to compute skb_csum_unnecessary
	     * will be on cold cache lines at recvmsg time.
	     * skb->len can be stored on 16 bits since the udp header has been
	     * already validated and pulled.
	     */
	     u16 len;
	     bool is_linear;
	     bool csum_unnecessary;
    #endif
    };

    static inline struct udp_dev_scratch *udp_skb_scratch(struct sk_buff *skb)
    {
	    return (struct udp_dev_scratch *)&skb->dev_scratch;
    }
    ```

    通过上面我们看到在`skb->dev_scrach`位置处存放了一个udp_dev_scratch结构.

### 2.1.1 sk_buff_head结构

在上面我们提到了`sk_buff_head`结构，其定义在include/linux/skbuff.h中:

```C
struct sk_buff_head {
	/* These two members must be first to match sk_buff. */
	struct_group_tagged(sk_buff_list, list,
		struct sk_buff	*next;
		struct sk_buff	*prev;
	);

	__u32		qlen;
	spinlock_t	lock;
};
```

由于`sk_buff`本身是一个链表结构，为了能够快速地从整个链表的头部找到每个sk_buff，因此在第一个sk_buff的前面会插入一个辅助头节点，即`sk_buff_head`。

1. **struct_group_tagged**

    `struct_group_tagged`是一个宏，定义在include/linux/stddef.hz中：

    ```C
    /**
     * struct_group_tagged() - Create a struct_group with a reusable tag
     *
     * @TAG: The tag name for the named sub-struct
     * @NAME: The identifier name of the mirrored sub-struct
     * @MEMBERS: The member declarations for the mirrored structs
     *
     * Used to create an anonymous union of two structs with identical
     * layout and size: one anonymous and one named. The former can be
     * used normally without sub-struct naming, and the latter can be
     * used to reason about the start, end, and size of the group of
     * struct members. Includes struct tag argument for the named copy,
     * so the specified layout can be reused later.
     */
    #define struct_group_tagged(TAG, NAME, MEMBERS...) \
	    __struct_group(TAG, NAME, /* no attrs */, MEMBERS)
    ```

    `struct_group_tagged`又引用了`__struct_group`宏，其定义在include/uapi/linux/stddef.h中:

    ```C
    /**
     * __struct_group() - Create a mirrored named and anonyomous struct
     *
     * @TAG: The tag name for the named sub-struct (usually empty)
     * @NAME: The identifier name of the mirrored sub-struct
     * @ATTRS: Any struct attributes (usually empty)
     * @MEMBERS: The member declarations for the mirrored structs
     *
     * Used to create an anonymous union of two structs with identical layout
     * and size: one anonymous and one named. The former's members can be used
     * normally without sub-struct naming, and the latter can be used to
     * reason about the start, end, and size of the group of struct members.
     * The named struct can also be explicitly tagged for layer reuse, as well
     * as both having struct attributes appended.
     */
    #define __struct_group(TAG, NAME, ATTRS, MEMBERS...) \
	    union { \
		    struct { MEMBERS } ATTRS; \
		    struct TAG { MEMBERS } ATTRS NAME; \
	    } ATTRS
    ```

    由此，展开之后我们得到如下结构:

    ```C
    union{
        struct {
            struct sk_buff	*next;
		    struct sk_buff	*prev;
        };
        struct sk_buff_list{
            struct sk_buff	*next;
		    struct sk_buff	*prev;
        } list;
    };
    ```

1. **qlen**

    表示链表中的节点数

1. **lock**

    用作多线程同步

1. **关键说明**

    `sk_buff` 和 `sk_buff_head` 开始的两个节点(next、prev)是相同的，即使 sk_buff_head 比 sk_buff 更轻量化.

## 2.2. 内存管理相关字段

1. **head**、**data**、**tail**、**end**

    - head: 指向数据缓冲区起始地址(通过`kmalloc`或页面分配)

    - data: 指向当前协议层有效数据的起始位置（如 TCP 报文头）。

    - tail: 指向当前有效数据的末尾

    - end: 指向数据缓冲区的结束位置

    
    **相关操作**: 通过`skb_reserve`预留头部空间，`skb_push/skb_pull` 调整 data 指针

1. **truesize**

    - 用途：记录 sk_buff 结构体及其数据缓冲区的总内存大小

    - 计算方式: `truesize = sizeof(struct sk_buff) + (end - head) + sizeof(struct skb_shared_info) + data_len`。

    关于socket buffer统计，更多详细信息可以参看[SKB socket accounting](http://oldvger.kernel.org/~davem/skb_sk.html)

1. **users**

    sk_buff线性空间(head~end)的原子引用计数，通过 skb_get 增加引用，kfree_skb 减少引用。

    关于`users`的使用场景，参看：Shared skbs and skb clones

## 2.3 数据包元数据

1. **len和data_len**

    这里要声明两个概念的区别，后续直接用这两个概念，注意区分：
      - 线性数据：`head - end`。

      - 实际线性数据：`data - tail`，不包含线性数据中的头空间和尾空间。

    接下来我们来看`len`和`data_len`两个变量:

      - len: skb中的数据块的总长度，数据块包括实际线性数据和非线性数据，非线性数据为data_len，所以skb->len= (data - tail) + data_len。

      - data_len: 仅包含非线性分片数据的长度（如通过 skb_shinfo(skb)->frags 管理的分页数据）


    因此`(len - data_len)`即为位于线性buffer 的数据大小，skb_headlen() 函数用于计算这个值：

    ```C
    static inline unsigned int skb_headlen(const struct sk_buff *skb)
    {
	    return skb->len - skb->data_len;
    }
    ```

1. **mac_len和hdr_len**

    - mac_len: 链路层头部长度（如以太网头的 14 字节）。

    - hdr_len: 保留字段，通常用于特定协议的自定义头部长度。

## 2.4 socket相关

```C
union {
    struct sock		*sk;
    int			ip_defrag_offset;
};
```

- sk: 指向与此skb关联的socket，当这个包是socket发出或接收时，这里指向对应的socket；而如果是转发包，这里是NULL。

- ip_defrag_offset: 用于 在IP分片重组过程中记录当前分片在完整数据包中的字节偏移量，以便内核能够正确合并所有分片数据


## 2.5 skb数据包发送与接收时间戳

```C
union {
    ktime_t		tstamp;
    u64		skb_mstamp_ns; /* earliest departure time */
};
```

- tstamp: 记录当前skb数据包发送或接收时的时间戳。由于计算时间戳代价昂贵，因此只会在必要的情况下进行记录。

- skb_mstamp_ns: 用于tcp重传，记录最早的tcp数据包重传时间


## 2.6 skb控制缓存区

```C
/*
    * This is the control buffer. It is free to use for every
    * layer. Please put your private variables there. If you
    * want to keep them across layers you have to do a skb_clone()
    * first. This is owned by whoever has the skb queued ATM.
    */
char			cb[48] __aligned(8);
```

- 用途：各协议层私有数据存储区（如 TCP 的 struct tcp_skb_cb）。

- 访问方式：通过宏 TCP_SKB_CB(skb) 访问 TCP 私有数据。

## 2.7 _skb_refdst、tcp_tsorted_anchor与_sk_redir

```C
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
```

1. **_skb_refdst及destructor**

    - **作用**：管理路由目标和清理回调。

      - `_skb_refdst`

        - 存储路由目标缓存（struct dst_entry）的引用信息，用于加速路由查找。

        - 通常通过原子操作管理引用计数（低位可能存储标志位，高位存储指针）。

      - `destructor`

        - 函数指针，指向释放 sk_buff 时调用的清理函数。

        - 例如，在数据包需要释放与路由相关的资源时，执行特定的回收逻辑。

    - **使用场景**：

      - 当数据包需要路由处理时（如发送或转发），内核通过 skb_dst_set() 设置 _skb_refdst。

      - 在释放 sk_buff 时，若设置了 destructor，则调用该函数释放额外资源。

1. **tcp_tsorted_anchor**

    - **作用:** 用于 TCP 协议中对乱序数据包进行排序
    
      - tcp_tsorted_anchor 是一个链表头，将需要按顺序重组的数据包链接起来

      - 在 TCP 接收路径中，若数据包乱序到达，内核会将其暂存到排序链表中，等待缺失的报文。

    - **使用场景**

      - 仅在 TCP 协议处理乱序数据包时生效。

      - 与 _skb_refdst 互斥：同一时刻，sk_buff 要么用于路由处理，要么用于 TCP 排序。

1. **_sk_redir**

    - **作用**：与 Socket 消息重定向（如 SO_INCOMING_CPU 或 eBPF 重定向）相关。

      - 存储重定向目标的信息（例如目标 CPU 核心或 Socket 标识）。

      - 用于优化多核环境下数据包的负载均衡或定向投递。

    - **使用场景**：

      - 当内核启用 CONFIG_NET_SOCK_MSG 配置时生效。

      - 在数据包需要绕过默认路径，直接投递到指定 CPU 或 Socket 时使用。


## 2.9 skb相关标志位

```C
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
```
- cloned：标记此 skb 是否为克隆（共享数据缓冲区）

- nohdr: 标记数据包头部是否已被剥离（如用于 AF_PACKET 原始套接字）。

- fclone: skb clone状态，可以取如下值

  ```C
  enum {
	  SKB_FCLONE_UNAVAILABLE,	/* skb has no fclone (from head_cache) */
	  SKB_FCLONE_ORIG,	/* orig skb (from fclone_cache) */
	  SKB_FCLONE_CLONE,	/* companion fclone skb (from fclone_cache) */
  };
  ```

- peeked: this packet has been seen already, so stats have been done for it, don't do them again

- head_frag: 表明SKB是从page fragment分配的，而不是通过kmalloc()或vmalloc()分配的

- pfmemalloc: 标记数据缓冲区是否从内存紧急分配池分配

- pp_recycle: 指示使用page pool来进行空间回收


## 2.9 skb headers结构

```C
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
```

### 2.9.1 struct_group结构

`struct_group`定义在include/uapi/linux/stddef.h中:

```C
/**
 * struct_group() - Wrap a set of declarations in a mirrored struct
 *
 * @NAME: The identifier name of the mirrored sub-struct
 * @MEMBERS: The member declarations for the mirrored structs
 *
 * Used to create an anonymous union of two structs with identical
 * layout and size: one anonymous and one named. The former can be
 * used normally without sub-struct naming, and the latter can be
 * used to reason about the start, end, and size of the group of
 * struct members.
 */
#define struct_group(NAME, MEMBERS...)	\
	__struct_group(/* no tag */, NAME, /* no attrs */, MEMBERS)

/**
 * __struct_group() - Create a mirrored named and anonyomous struct
 *
 * @TAG: The tag name for the named sub-struct (usually empty)
 * @NAME: The identifier name of the mirrored sub-struct
 * @ATTRS: Any struct attributes (usually empty)
 * @MEMBERS: The member declarations for the mirrored structs
 *
 * Used to create an anonymous union of two structs with identical layout
 * and size: one anonymous and one named. The former's members can be used
 * normally without sub-struct naming, and the latter can be used to
 * reason about the start, end, and size of the group of struct members.
 * The named struct can also be explicitly tagged for layer reuse, as well
 * as both having struct attributes appended.
 */
#define __struct_group(TAG, NAME, ATTRS, MEMBERS...) \
	union { \
		struct { MEMBERS } ATTRS; \
		struct TAG { MEMBERS } ATTRS NAME; \
	} ATTRS
```
因此，这里针对`sk_buff`展开即为：

```C
union{
    struct{ MEMBERS};
    struct { MEMBERS } headers;
}
```

通过上面可知，struct_group创建了一个匿名的union。

### 2.9.2 关键字段分析

1. **pkt_type**

    用于标识数据包类型。取值可以为(include/uapi/linux/if_packet.h):

    - PACKET_HOST: 表示发给本机的数据包

    - PACKET_BROADCAST：广播包

    - PACKET_MULTICAST：组播包

    - PACKET_OTHERHOST: 发给其他主机的包

    - PACKET_OUTGOING：outgoing的任何数据包

    - PACKET_USER：发送给用户空间的数据包

    - PACKET_KERNEL: 发送给内核空间的数据包

1. **ignore_df**

    用于 控制是否忽略IP头中的“Don’t Fragment”（DF）标志位，允许内核在特定场景下强制对数据包进行分片（即使DF位被标记）。

    **核心作用:**

      - IP头中的 DF 标志位通常用于指示数据包是否允许分片（例如，用于路径MTU发现）

      - 当 ignore_df 字段设置为 1 时，内核会忽略IP头中的 DF 标志，强制对数据包进行分片（即使原IP头标记了 DF=1）。

    **典型应用场景:**

      - 隧道封装（如IPsec、GRE）：外层IP包可能需要分片，而内层IP包的 DF 标志应被忽略

      - 透明代理或NAT：修改数据包后需要重新分片，但需保留原始 DF 标志状态。

      - 调试或特殊网络配置：强制分片以测试网络路径的MTU兼容性

1. **ip_summed**

    - 用途：指示校验和状态，可选值：

      - CHECKSUM_NONE：未计算校验和。

      - CHECKSUM_COMPLETE：硬件已计算 L4 校验和。

      - CHECKSUM_PARTIAL：需要硬件补全校验和。

1. **ooo_okay**

    指示是否允许socket所映射到的队列发生改变

1. **encapsulation、encap_hdr_csum、csum_valid**

    ```C
    /* Indicates the inner headers are valid in the skbuff. */
	__u8			encapsulation:1;
	__u8			encap_hdr_csum:1;
	__u8			csum_valid:1;
    ```

    - encapsulation: 用于指示sk_buff的inner headers是否有效

    - encap_hdr_csum: 用于指示需要checksum

    - csum_valid: 当值为1时，表示checksum已经计算过，已经有效了

1. **alloc_cpu**

    指示当前skb是哪一个CPU分配的

1. **校验和相关**

    ```C
    union {
		__wsum		csum;
		struct {
			__u16	csum_start;
			__u16	csum_offset;
		};
	};
    ```

    - csum: 存储校验和计算结果。

    - csum_start: 从skb->head指定偏移处开始计算checksum

    - csum_offset: 指定将checksum保存到从csum_start位置起的offset处

1. **priority**

    指定数据包的queueing优先级

1. **skb_iif**

    指定skb数据包从哪一个设备接口编号(interface index)发送，或从哪一个设备接口编号接收的

1. **hash**

    skb数据包hash值

1. **vlan相关**

    ```C
    union {
		u32		vlan_all;
		struct {
			__be16	vlan_proto;
			__u16	vlan_tci;
		};
	};
    ```

    - vlan_proto: vlan所封装的协议

    - vlan_tci: vlan tag控制信息

1. **协议头指针相关**

    **mac_header、network_header、transport_header**

      - mac_header: 指向链路层（L2）头部（如以太网头）

      - network_header: 指向网络层（L3）头部（如 IP 头）

      - transport_header: 指向传输层（L4）头部（如 TCP/UDP 头）。

      - 操作: 通过 skb_reset_mac_header、skb_set_network_header 等函数设置

    **inner_transport_header、inner_network_header、inner_mac_header**

      - 用途：用于隧道协议（如 VXLAN、GRE），指向内部协议头。

      - 示例：inner_transport_header 指向隧道内部传输层头。


    **protocol**

      - 用途: 标识数据包的协议类型（如 ETH_P_IP 表示 IPv4 包），可以在/usr/include/linux/if_ether.h找到protocol的相关定义

        ```C
        #define ETH_P_LOOP	0x0060		/* Ethernet Loopback packet	*/
        #define ETH_P_PUP	0x0200		/* Xerox PUP packet		*/
        #define ETH_P_PUPAT	0x0201		/* Xerox PUP Addr Trans packet	*/
        #define ETH_P_TSN	0x22F0		/* TSN (IEEE 1722) packet	*/
        #define ETH_P_ERSPAN2	0x22EB		/* ERSPAN version 2 (type III)	*/
        #define ETH_P_IP	0x0800		/* Internet Protocol packet	*/
        ```

      - 设置时机: 由 eth_type_trans 在接收路径中根据以太网头的 type 字段设置


    **inner_protocol、inner_ipproto**

      - 用途：用于隧道协议，标识内部数据包的协议类型

## 2.7 其他字段

1. **_nfct**

    ```C
    #if defined(CONFIG_NF_CONNTRACK) || defined(CONFIG_NF_CONNTRACK_MODULE)
	unsigned long		 _nfct;
    #endif
    ```

    用于保存netfilter的连接跟踪信息。


1. **queue_mapping**

    对于多队列设备，用于指定当前skb数据包采用哪一个队列进行发送，或者从哪一个队列接收的。

    
# 3. Shared skbs and skb clones

## 3.1 共享skb

`sk_buff.users`是一个引用计数，允许多个实体(entity)共享一个sk_buff。当`sk_buff.users != 1`时，我们称该skb为共享skb（ps： 这有点类似于C++中的shared_ptr)。

>ps: 这里是共享线性buffer部分

下面我们分别介绍sk_buff.users的初始值、增加引用计数、减少引用计数的相关操作。


1. **sk_buff.users初始值**

    在分配skb数据结构的时候将`sk_buff.users`的值设置为了1，参看如下:

    ```C
    /**
     * alloc_skb - allocate a network buffer
     * @size: size to allocate
     * @priority: allocation mask
     *
     * This function is a convenient wrapper around __alloc_skb().
     */
    static inline struct sk_buff *alloc_skb(unsigned int size,
					gfp_t priority)
    {
	    return __alloc_skb(size, priority, 0, NUMA_NO_NODE);
    }

    struct sk_buff *__alloc_skb(unsigned int size, gfp_t gfp_mask,
			    int flags, int node)
    {
        ...

        memset(skb, 0, offsetof(struct sk_buff, tail));
	    __build_skb_around(skb, data, size);

        ...
    }

    /* Caller must provide SKB that is memset cleared */
    static void __build_skb_around(struct sk_buff *skb, void *data,
			       unsigned int frag_size)
    {
	    unsigned int size = frag_size;

	    /* frag_size == 0 is considered deprecated now. Callers
	     * using slab buffer should use slab_build_skb() instead.
	     */
	    if (WARN_ONCE(size == 0, "Use slab_build_skb() instead"))
		    data = __slab_build_skb(skb, data, &size);

	    __finalize_skb_around(skb, data, size);
    }

    static inline void __finalize_skb_around(struct sk_buff *skb, void *data,
					 unsigned int size)
    {
        struct skb_shared_info *shinfo;
        
	    size -= SKB_DATA_ALIGN(sizeof(struct skb_shared_info));
        
	    /* Assumes caller memset cleared SKB */
	    skb->truesize = SKB_TRUESIZE(size);
	    refcount_set(&skb->users, 1);

        ...
    }
    ```

1. **skb_get()增加引用计数**

    ```C
    /**
     *	skb_get - reference buffer
     *	@skb: buffer to reference
     *
     *	Makes another reference to a socket buffer and returns a pointer
     *	to the buffer.
     */
    static inline struct sk_buff *skb_get(struct sk_buff *skb)
    {
	    refcount_inc(&skb->users);
	    return skb;
    }
    ```

1. **kfree_skb()减少引用计数**

    ```C
    /**
     *	kfree_skb - free an sk_buff with 'NOT_SPECIFIED' reason
     *	@skb: buffer to free
     */
    static inline void kfree_skb(struct sk_buff *skb)
    {
    	kfree_skb_reason(skb, SKB_DROP_REASON_NOT_SPECIFIED);
    }

    void __fix_address
    kfree_skb_reason(struct sk_buff *skb, enum skb_drop_reason reason)
    {
	    if (__kfree_skb_reason(skb, reason))
	    	__kfree_skb(skb);
    }

    static __always_inline
    bool __kfree_skb_reason(struct sk_buff *skb, enum skb_drop_reason reason)
    {
	    if (unlikely(!skb_unref(skb)))
		    return false;

        ...
    }
    /**
     * skb_unref - decrement the skb's reference count
     * @skb: buffer
     *
     * Returns true if we can free the skb.
     */
    static inline bool skb_unref(struct sk_buff *skb)
    {
	    if (unlikely(!skb))
		    return false;
	    if (likely(refcount_read(&skb->users) == 1))
		    smp_rmb();
	    else if (likely(!refcount_dec_and_test(&skb->users)))
		    return false;

	    return true;
    }
    ```

## 3.2 skb克隆

skb_clone()可以快速的复制一个skb。这里的`复制`并不会真正复制sk_buffer的data buffer部分，而仅仅是复制`struct sk_buff`的元数据结构。`&skb_shared_info.refcount`表明了指向同一个packet的skb数量。

skb_clone()也会造成`sk_buff.users`的数量增加1，下来我们来看看:

```C
/*
 * You should not add any new code to this function.  Add it to
 * __copy_skb_header above instead.
 */
static struct sk_buff *__skb_clone(struct sk_buff *n, struct sk_buff *skb)
{
#define C(x) n->x = skb->x

	n->next = n->prev = NULL;
	n->sk = NULL;
	__copy_skb_header(n, skb);

	C(len);
	C(data_len);
	C(mac_len);
	n->hdr_len = skb->nohdr ? skb_headroom(skb) : skb->hdr_len;
	n->cloned = 1;
	n->nohdr = 0;
	n->peeked = 0;
	C(pfmemalloc);
	C(pp_recycle);
	n->destructor = NULL;
	C(tail);
	C(end);
	C(head);
	C(head_frag);
	C(data);
	C(truesize);
	refcount_set(&n->users, 1);

	atomic_inc(&(skb_shinfo(skb)->dataref));
	skb->cloned = 1;

	return n;
#undef C
}
```

# 4. skb_shared_info介绍

skb_shared_info部分保存着一个packet的分片信息。下面我们来看看该数据结构:

```C
/* This data is invariant across clones and lives at
 * the end of the header data, ie. at skb->end.
 */
struct skb_shared_info {
	__u8		flags;
	__u8		meta_len;
	__u8		nr_frags;
	__u8		tx_flags;
	unsigned short	gso_size;
	/* Warning: this field is not always filled in (UFO)! */
	unsigned short	gso_segs;
	struct sk_buff	*frag_list;
	struct skb_shared_hwtstamps hwtstamps;
	unsigned int	gso_type;
	u32		tskey;

	/*
	 * Warning : all fields before dataref are cleared in __alloc_skb()
	 */
	atomic_t	dataref;
	unsigned int	xdp_frags_size;

	/* Intermediate layers must ensure that destructor_arg
	 * remains valid until skb destructor */
	void *		destructor_arg;

	/* must be last field, see pskb_expand_head() */
	skb_frag_t	frags[MAX_SKB_FRAGS];
};
```

参看如下文章:

- [内核 skb/sk_buff 详解](https://zhuanlan.zhihu.com/p/626514905)

- [Linux sk_buff结构](https://www.cnblogs.com/ink-white/p/16814624.html)

- [struct sk_buff](https://lishiwen4.github.io/network/sk_buff)

- [skbuff pdf](https://people.computing.clemson.edu/~westall/853/notes/skbuff.pdf)

## 4.1 关键字段解析


  

# 5. SKB数据区域相关操作

skb数据区域的相关操作参看: http://oldvger.kernel.org/~davem/skb_data.html



# 6. 总结

struct sk_buff 是 Linux 网络协议栈的核心枢纽，其字段设计紧密围绕以下目标：

- 高效内存管理：通过指针调整避免数据拷贝。

- 跨协议层协作：通过分层头指针支持协议栈处理。

- 硬件加速支持：通过标志位和字段（如 ip_summed）实现硬件卸载。

- 动态扩展性：通过 cb 和 skb_ext 支持私有数据存储。

理解每个字段的用途及操作函数（如 skb_push、skb_clone）是开发内核网络模块或优化协议栈性能的关键基础
