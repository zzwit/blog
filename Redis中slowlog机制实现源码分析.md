Slowlog是Redis从2.2.12版本引入的一条命令，用于查询和重置Redis内部维护的慢查询日志，其中每条日志的内容均由四部分组成：1）慢查询日志Id，该Id对于每条日志来说都是唯一的；2）慢查询日志被记录的Unix时间戳；3）慢查询的执行时间，以微秒计；4）查询本身，包括命令以及一系列参数。
    需要说明的是，所谓慢查询指的就是内部执行时间超过某个指定时限的查询，而控制该指定时限的就是Redis配置文件中的配置项slowlog-log-slower-than。除了slowlog-log-slower-than外，在配置文件中还有另外一个参数与慢查询日志有关，那就是slowlog-max-len，该配置项控制了Redis系统最多能够维护多少条慢查询日志。因为虽然我们这里称之为日志，但实际上它们仅维护在内存中而不会写出到磁盘的日志文件上去，所以slowlog-max-len的作用就是控制慢查询日志的内存占有量。
    Slowlog可用的命令形式有三种：1）slowlog get [number]，返回最近的number条慢查询日志，如果不提供number参数则返回全部慢查询日志；2）slowlog len，返回当前已有慢查询日志的条数；3）slowlog reset，清空当前所有的慢查询日志。
    以上对Slowlog进行了简单的介绍，接下来我们就通过源码来看一看，Redis的slowlog机制是如何实现的。实现slowlog的源码文件非常好找，那就是slowlog.h和slowlog.c，首先来看头文件slowlog.h。代码段一给出了slowlog.h的全部内容，可以看到其在最开始部分定义了两个宏。其中，SLOWLOG_ENTRY_MAX_ARGC定义了每条慢查询日志记录查询参数的最大个数为32个（譬如用户执行一条MGET命令，后面跟了100个key，完整地记录下该查询显然会占用很多空间，根据SLOWLOG_ENTRY_MAX_ARGC的限制，Redis只会记录MGET和前面的31个key）。SLOWLOG_ENTRY_MAX_STRING则定义了慢查询日志中每个查询参数最长只记录128个字符（譬如用户执行一条GET命令，后面跟一个长度为1024的key，那么对不起，Redis只会记录这个key的前128个字符，后面的全部截断）。由此不难看出，Redis作者在如何避免内存浪费方面真的是花了很多心思。

代码段一

#define SLOWLOG_ENTRY_MAX_ARGC 32
#define SLOWLOG_ENTRY_MAX_STRING 128

typedef struct slowlogEntry {
    robj **argv;
    int argc;
    long long id;
    long long duration;
    time_t time;
} slowlogEntry;

void slowlogInit(void);
void slowlogPushEntryIfNeeded(robj **argv, int argc, long long duration);
void slowlogCommand(redisClient *c);

    除了两个宏定义外，slowlog.h还给出了系统内部如何保存慢日志查询的结构体slowlogEntry。除了两个宏定义外，slowlog.h还给出了系统内部如何保存慢查询日志的结构体slowlogEntry。在该结构体中，argv和argc用于保存慢查询的参数和参数个数，id就是日志的唯一Id，duration顾名思义就是该查询的执行时间，而time就是该日志被记录时的Unix时间戳。
    最后，slowlog.h还声明了三个对外的函数接口。首先，slowlogInit用于初始化慢查询日志，该函数只在一个地方进行了调用，那就是初始化Redis服务器的initServer函数中。其次，slowlogPushEntryIfNeeded用于判断一条刚刚执行完的查询是否要划归为慢查询的行列，如果是就为其记录一条慢查询日志。该函数同样只在一个地方调用，那就是用于执行所有Redis命令的call函数，该函数执行完每条命令之后都会计算一下执行时间，然后调用slowlogPushEntryIfNeeded函数记录可能的慢查询日志。最后，slowlogCommand用于处理slowlog命令，它会检查slowlog命令的语法、执行命令并向客户端返回执行结果。
    看完了slowlog.h之后，我们再来看看slowlog.c中的内容，为了便于介绍，我们不妨就从slowlogInit函数开始。代码段二给出了slowlogInit函数的代码实现，从中我们可以知道存储了慢查询日志的就是全局变量server中的一个成员slowlog，而该成员本身就是一个list，所以初始化慢查询日志其实就是创建一个list，该list中的元素类型就是上面的slowlogEntry。代码段二中同时给出了list的释放方法，用以在删除list元素时释放元素所占用的内存空间。

代码段二

void slowlogInit(void) {
    创建一个list并赋值给server.slowlog
    server.slowlog = listCreate();
    将server.slowlog_entry_id赋值为0，它就是慢查询日志Id的生成器，每创建一条慢查询日志，
    该值就会加一
    server.slowlog_entry_id = 0;
    设定慢查询日志list的释放方法，用来释放list中的元素，该list中的元素也就是代码段一中给出
    的slowlogEntry
    listSetFreeMethod(server.slowlog,slowlogFreeEntry);
}

void slowlogFreeEntry(void *septr) {
    slowlogEntry *se = septr;
    int j;
    将slowlogEntry中保存的所有查询参数的引用计数减一，如果引用计数降到零，
    查询参数占用的内存会被自动释放
    for (j = 0; j < se->argc; j++)
        decrRefCount(se->argv[j]);
    释放查询参数指针数组的空间
    zfree(se->argv);
    释放slowlogEntry自身的空间
    zfree(se);
}

    看完了slowlogInit函数的相关实现后，我们再接着看slowlogPushEntryIfNeeded函数。代码段三给出了它的代码实现，其原理就是检查给定查询的执行时间，如果符合慢查询的条件就为其创建一条慢查询日志并添加到server.slowlog中。在添加完新的慢查询日志后，如果出现list长度超出限制的情况，就删除旧的慢查询日志。代码段三同时给出了用于创建一条慢查询日志的slowlogCreateEntry的函数实现。

代码段三

void slowlogPushEntryIfNeeded(robj **argv, int argc, long long duration) {
    检查参数slowlog_log_slower_than的值，如果该值小于0说明慢查询机制被关闭了，
    慢查询日志也就无需记录了，直接返回即可
    if (server.slowlog_log_slower_than < 0) return;
    判断查询的执行时间是否比slowlog_log_slower_than大，如果是则为该查询创建一个
    slowlogEntry并将其添加到server.slowlog这个list的头部，也就是说越晚记录的慢
    查询在list中就越靠前
    if (duration >= server.slowlog_log_slower_than)
        listAddNodeHead(server.slowlog,slowlogCreateEntry(argv,argc,duration));
    检查当前server.slowlog的长度是否已经超过slowlog_max_len的值了，如果是则删除
    server.slowlog尾部的list元素，直到其长度满足要求
    while (listLength(server.slowlog) > server.slowlog_max_len)
        listDelNode(server.slowlog,listLast(server.slowlog));
}

slowlogEntry *slowlogCreateEntry(robj **argv, int argc, long long duration) {
    为slowlogEntry结构分配内存空间
    slowlogEntry *se = zmalloc(sizeof(*se));
    int j, slargc = argc;
    检查给定查询的参数个数，如果其超过了SLOWLOG_ENTRY_MAX_ARGC的限制，则将真实
    记录的参数个数强制设定为SLOWLOG_ENTRY_MAX_ARGC
    if (slargc > SLOWLOG_ENTRY_MAX_ARGC) slargc = SLOWLOG_ENTRY_MAX_ARGC;
    se->argc = slargc;
    为参数指针数组分配空间
    se->argv = zmalloc(sizeof(robj*)*slargc);
    将给定参数的前slargc-1个添加到参数指针数组中
    for (j = 0; j < slargc; j++) {
        如果当前的参数是要记录的最后一个参数而且给定参数无法全部记录在日志中，
       那么这最后一个参数会记录我们舍弃了多少个给定参数没有记录，感觉此处的
       设计还是很贴心的，虽然Redis不会对所有给定参数照单全收，但是我会告诉你
       究竟有几个参数被忽略了
        if (slargc != argc && j == slargc-1) {
            se->argv[j] = createObject(REDIS_STRING,
                sdscatprintf(sdsempty(),"... (%d more arguments)",
                argc-slargc+1));
        } else {
            如果当前的参数不是要记录的最后一个或者没有参数被忽略，那么要检查该
          参数是不是未经任何编码的字符串类型，如果是而且其长度还超过了 
          SLOWLOG_ENTRY_MAX_STRING的限制，那么就要截断超长的字符串
            if (argv[j]->type == REDIS_STRING &&
                argv[j]->encoding == REDIS_ENCODING_RAW &&
                sdslen(argv[j]->ptr) > SLOWLOG_ENTRY_MAX_STRING)
            {
                此处对于字符串的截断同样会附加有多少字符串被截断的提示信息
                sds s = sdsnewlen(argv[j]->ptr, SLOWLOG_ENTRY_MAX_STRING);
                s = sdscatprintf(s,"... (%lu more bytes)",
                    (unsigned long)
                    sdslen(argv[j]->ptr) - SLOWLOG_ENTRY_MAX_STRING);
                se->argv[j] = createObject(REDIS_STRING,s);
            } else {
                如果无需进行截断就直接将参数指针添加到数组中并将其引用计数加一，
              以防止该参数在别处被释放
                se->argv[j] = argv[j];
                incrRefCount(argv[j]);
            }
        }
    }
    设置日志的添加时间戳、日志Id和查询的执行时间
    se->time = time(NULL);
    se->duration = duration;
    se->id = server.slowlog_entry_id++;
    return se;
}

    看完了slowlogPushEntryIfNeeded的相关实现，我们最后再来看看slowlogCommand。代码段四给出了该函数的代码实现，它首先会对命令语法进行检查并判断出用户要执行的是三种命令形式中的哪一种。对于slowlog reset命令，直接调用函数slowlogReset将server.slowlog中的所有元素清空。对于slowlog len，直接向客户端返回server.slowlog的list长度。对于slowlog get [number]，则是将server.slowlog中的前number条日志按照记录时间由新到旧的顺序返回给客户端。

代码段四

void slowlogCommand(redisClient *c) {
    命令参数个数为2，且第二个参数是reset，确定用户执行的是slowlog reset命令，
   直接清空server.slowlog中的全部元素，然后向客户端返回OK
    if (c->argc == 2 && !strcasecmp(c->argv[1]->ptr,"reset")) {
        slowlogReset();
        addReply(c,shared.ok);
    }
    命令参数个数为2，且第二个参数是len，确定用户执行的是slowlog len命令，
    直接向客户端返回server.slowlog的list长度 
    else if (c->argc == 2 && !strcasecmp(c->argv[1]->ptr,"len")) {
        addReplyLongLong(c,listLength(server.slowlog));
    }
    命令参数个数为2或3，且第二个参数是get，确定用户执行的是slowlog get [number]命令
    else if ((c->argc == 2 || c->argc == 3) &&
               !strcasecmp(c->argv[1]->ptr,"get"))
    {
        long count = 10, sent = 0;
        listIter li;
        void *totentries;
        listNode *ln;
        slowlogEntry *se;
        命令参数个数为3，所以存在第三个参数number，将其转换为long型值，
       如果参数number不合法无法转换为long型值，getLongFromObjectOrReply函数
       会向客户端报错错误信息，本函数直接返回即可
        if (c->argc == 3 &&
            getLongFromObjectOrReply(c,c->argv[2],&count,NULL) != REDIS_OK)
            return;
        获取server.slowlog的一个迭代器用来遍历list中的元素，待迭代器起始位置为list头部
        listRewind(server.slowlog,&li);
        totentries = addDeferredMultiBulkLength(c);
        遍历并返回server.slowlog中的慢查询日志，直到满足用户指定的查询个数或者遍历至
       list末尾
        while(count-- && (ln = listNext(&li))) {
            int j;
            se = ln->value;
            addReplyMultiBulkLen(c,4);
            依次返回日志Id、日志记录时间戳、查询执行时间和具体查询参数
            addReplyLongLong(c,se->id);
            addReplyLongLong(c,se->time);
            addReplyLongLong(c,se->duration);
            addReplyMultiBulkLen(c,se->argc);
            for (j = 0; j < se->argc; j++)
                addReplyBulk(c,se->argv[j]);
            sent++;
        }
        setDeferredMultiBulkLength(c,totentries,sent);
    }
    无法解析查询命令，向客户端返回报错信息 
    else {
        addReplyError(c,
            "Unknown SLOWLOG subcommand or wrong # of args. Try GET, RESET, LEN.");
    }
}

void slowlogReset(void) {
    不停地删除server.slowlog末尾的元素，直到整个list的长度为零
    while (listLength(server.slowlog) > 0)
        listDelNode(server.slowlog,listLast(server.slowlog));
}

    以上就是关于slowlog机制实现的全部代码，其中代码本身没有任何难懂的地方（甚至可以说是Redis所有机制中最容易读懂的部分），给我体会最深的还是作者处处都在节省内存的良苦用心。最后还需要补充的是，本博文中展示的代码均来自Redis 2.6.12版本。

参考文献链接：http://redis.io/commands/slowlog
