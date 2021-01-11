# Redis-string存储结构

## 1.redis底层存储字符串的方式：

​	  	redis虽然由c语言编写，但并没有像c语言一样直接使用字符数组存储字符串，而是将字符数组封装成一个称为简单动态字符串的结构体；即：simple dynamic string，简称为SDS；

​		sds是一个由四个成员组成的结构体，结构体名称为sdshdr，其四个成员分别是:

​			1.表示字符串实际长度的len；

​			2.存储字符串内容的字符数组 buf；

​			3.字符数组容量 alloc；

​			4.sdshdr类型 flags；

​		并且redis为了进一步优化数据占用的内存空间，使用不同位域大小的成员构成的结构体来存储字符串，其中结构体中的flags成员即表示当前sdshdr的数据类型；具体情况如下：

```c
/* sdshdr5实际上并没有使用 */
struct __attribute__ ((__packed__)) sdshdr5 {
  unsigned char flags; /* 前3位用来存储sds类型，后5位暂时不用 */
  char buf[]; /* 存储字符串的字符数组 */
};

/* redis存储字符串时实际上以sdshdr8开始 */
struct __attribute__ ((__packed__)) sdshdr8 {
  uint8_t len; /* 字符串实际长度 */
  uint8_t alloc; /* 字符数组容量 */                             
  unsigned char flags; /* 前3位用来存储sds类型，后5位暂时不用 */
  char buf[]; /* 存储字符串的字符数组 */
};
struct __attribute__ ((__packed__)) sdshdr16 {
  uint16_t len; /* 字符串实际长度 */
  uint16_t alloc; /* 字符数组容量 */
  unsigned char flags; /* 前3位用来存储sds类型，后5位暂时不用 */
  char buf[]; /* 存储字符串的字符数组 */
};
struct __attribute__ ((__packed__)) sdshdr32 {
  uint32_t len; /* used */
  uint32_t alloc; /* excluding the header and null terminator */
  unsigned char flags; /* 3 lsb of type, 5 unused bits */
  char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
  uint64_t len; /* used */
  uint64_t alloc; /* excluding the header and null terminator */
  unsigned char flags; /* 3 lsb of type, 5 unused bits */
  char buf[];
};
```



## 2.SDS的创建、扩容和销毁方式：

SDS的创建过程流程：

​	1.根据字符串长度确定存储该字符串的sdshdr结构体类型；

​	2.计算sdshdr的大小，并根据该大小申请内存空间；	

​	3.初始化为sdshdr申请的内存空间为0；

​	4.根据sdshdr的类型来初始化sdshdr结构体中的各项成员；

​	5.将字符串内容拷贝至结构体中的字符数组中，并返回新创建的sdshdr结构体；

SDS创建过程源码：

```c
//redis中通过sdsnewlen函数创建sds字符串，该函数在sds.c中定义
sds sdsnewlen(const void *init, size_t initlen) {
    void *sh;
    sds s;
    /* 根据字符串初始长度确定需要使用的sds类型 */
    char type = sdsReqType(initlen);
    /* 空字符串会使用sds8 */
    if (type == SDS_TYPE_5 && initlen == 0) type = SDS_TYPE_8;
    /* 计算当前sdshdr大小 */
    int hdrlen = sdsHdrSize(type);
    unsigned char *fp; 
    /* 为sdshdr申请内存，+1表示字符串结束符 */
    sh = s_malloc(hdrlen+initlen+1);
    if (!init)
      	/* 初始化sds内存空间 */
        memset(sh, 0, hdrlen+initlen+1);
    if (sh == NULL) return NULL;
    /* 获取数据部分起始指针 */
    s = (char*)sh+hdrlen;
    /* 此时buf还是空，所以s-1得到flags指针 */
    fp = ((unsigned char*)s)-1;
    /* 根据sds类型初始化sdshdr */
    switch(type) {
        case SDS_TYPE_5: {
            *fp = type | (initlen << SDS_TYPE_BITS);
            break;
        }
        case SDS_TYPE_8: {
            SDS_HDR_VAR(8,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_16: {
            SDS_HDR_VAR(16,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_32: {
            SDS_HDR_VAR(32,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
        case SDS_TYPE_64: {
            SDS_HDR_VAR(64,s);
            sh->len = initlen;
            sh->alloc = initlen;
            *fp = type;
            break;
        }
    }
    if (initlen && init)
        /* 将字符串内容拷贝至内存中 */
        memcpy(s, init, initlen);
  	/* 与c字符串兼容，使用\0作为字符串结尾 */
    s[initlen] = '\0'; 
  	/* 返回新创建的sds */
    return s; 
}
```

SDS扩容流程：

​		在向sds中追加内容时会触发sds的扩容机制，其扩容流程如下：

​		1.计算追加字符串之后所需要的字符数组大小；

​		2.检测当前字符数组剩余容量是否能够容纳要追加的字符串，如果能够容纳的话则直接返回，否则执行扩容流程；

​		3.在扩容之前会先检测追加之后的新字符串所需长度是否超过SDS_MAX_PREALLOC，如果超过则将SDS_MAX_PREALLOC所需长度作为新字符数组的长度返回；否则将原字符数组扩容为原来的2倍；

​		4.扩容之后字符串长度变长，因此原来的sdshdr结构体可能已经不再适用，所以，扩容之后还会检测当前sdshdr结构体是否仍然适用，如果仍然适用，则直接将字符数组realloc即可，否则还需要确定扩容之后的字符串所需要的sdshdr类型。并将原sds中的内容拷贝至新sds，并销毁原sds；

SDS扩容源码如下：

```c
//redis的SDS 通过sdsMakeRoomFor来扩容
//s表示当前需要被扩容的sds，addlen表示需要增加的长度
sds sdsMakeRoomFor(sds s, size_t addlen) {
    void *sh, *newsh;
    /* 计算当前sds剩余容量 */
    size_t avail = sdsavail(s);
    size_t len, newlen;
    char type, oldtype = s[-1] & SDS_TYPE_MASK;
    int hdrlen;
    /* 剩余容量大于待增加容量时无需扩容*/
    if (avail >= addlen) return s;
		
  	/* 返回当前len成员值，即字符串长度 */
    len = sdslen(s);
    sh = (char*)s-sdsHdrSize(oldtype);
  	/* 计算容纳追加内容之后的新长度 */
    newlen = (len+addlen);
    /* 如果所需的长度小于SDS_MAX_PREALLOC，将扩容后的容量设置为所需长度的2倍*/
    if (newlen < SDS_MAX_PREALLOC)
        newlen *= 2;
    else
        /* 否则将扩容后的容量设置为所需长度加上SDS_MAX_PREALLOC */
        newlen += SDS_MAX_PREALLOC;
		
  	/* 扩容后字符串长度变长，可能无法再使用原来的SDS header，因此需根据新的长度获取扩容之后需要使用的SDS header */
    type = sdsReqType(newlen);
    /* 如果sds的类型是SDS_TYPE_5，修改为SDS_TYPE_8*/
    if (type == SDS_TYPE_5) type = SDS_TYPE_8;
		/*计算扩容之后sdshdr大小*/
    hdrlen = sdsHdrSize(type);
    if (oldtype==type) {
        /* 如果扩容之后sdshdr类型未变，直接realloc，将原来的内存空间扩大即可*/
        newsh = s_realloc(sh, hdrlen+newlen+1);
        if (newsh == NULL) return NULL;
        s = (char*)newsh+hdrlen;
    } else {
        /*如果扩容之后sdshdr类型发生改变，需要重新申请内存，将原sdshdr内容拷贝至新的内存，并释放掉原sdshdr占用的内存*/
      /*申请新的内存*/
        newsh = s_malloc(hdrlen+newlen+1);
        if (newsh == NULL) return NULL;
      /* 将原来的sds拷贝至新的存储空间中 */
        memcpy((char*)newsh+hdrlen, s, len+1);
      /*释放原来sds所占的空间*/
        s_free(sh);
        s = (char*)newsh+hdrlen;
        s[-1] = type;
        /*重新设置len*/
        sdssetlen(s, len);
    }
    sdssetalloc(s, newlen);
    /* 返回扩容之后的sds */
    return s;
}
```

销毁：直接将sds占用的内存空间释放即可；

```c
void sdsfree(sds s) {
    if (s == NULL) return;
    s_free((char*)s-sdsHdrSize(s[-1]));
}
```



## 3.SDS在redis中的使用场景：

​		redis中所有使用到字符串的场景都是使用sds来存储的：

​			**1.redis中的键名都是使用sds的方式存储的；**

​			**2.redis中其他数据结构中的字符串类型的数据也是通过sds的方式存储的；**



## 4.sds和字符数组特点比较：

> **1.sds时二进制安全的，而使用字符数组存储字符串的方式非二进制安全**

​		使用字符数组存储字符串时会为字符串末尾添加 '\0' 来表示字符串的结尾，如果要存储的信息本身就有一个 '\0' 字符，那么这个字符在c语言中会被当做字符串的结尾，最终导致 '\0'之后的内容被截断，造成字符串错误；

​		而sds中并不会以 '\0' 作为字符串的结尾，而是把 '\0' 当做一个普通的字符存储，字符串的结尾由结构体中的len成员决定；获取字符串时直接从buf中获取len个字符即可，进而保证二进制安全；

> **2.获取字符串长度的效率更高**

​		使用字符数组存储字符串时，必须通过遍历字符数组查找字符串的结尾 '\0' 来确定字符串的字符个数，时间复杂度时O(n);

​		而使用sds存储字符串时，无需查找字符串结尾，字符串的字符个数有成员len存储，直接返回len的值即可，时间复杂度时O(1)；

> **3.可以有效防止缓冲区溢出，并且扩容效率更高**

​		使用字符数组存储字符串时，并不知道当前剩余容量是否能够容纳新增字符，强行添加可能发生缓冲区溢出错误，如果需要对字符数组扩容，也必须先遍历计算当前字符数组剩余容量，然后计算需要扩展的容量；

​		而使用sds存储字符串时无需计算，字符数组剩余容量可以由成员len和成员alloc计算得出，可以很快判断出当前字符数组是否需要扩容，需要扩展的容量也可以直接计算的出，因此效率更高；

> **4.sds相比字符数组会占用更多存储空间**

​		sds也并非全是优势，sds除过存储字符串以外还会存储其他信息，因此相比直接使用字符数组要消耗更多的内存；



## 5.string存储结构的创建过程：

​		redis中的string数据数据结构表示向redis中添加一个字符串类型的数据；而redis中的字符串虽然都是通过sds来存储的，但是对于不同类型、不同长度的字符串会使用不同的方式来实现sds，也就是字符串在底层的编码方式并不相同，有三种编码方式：OBJ_ENCODING_RAW、OBJ_ENCODING_EMBSTR和OBJ_ENCODING_INT

​		如果string存储结构存储的数据是一个数字字符串，且范围在LONG_MIN和LONG_MAX之间那么会直接使用OBJ_ENCODING_INT作为robj的encoding,，也就是直接将该数字字符串存储为一个整数，并将redisObject中的ptr指向该整数的地址；

```c
robj *createStringObjectFromLongLong(long long value) {
    robj *o;
    if (value >= 0 && value < OBJ_SHARED_INTEGERS) {
        incrRefCount(shared.integers[value]);
        o = shared.integers[value];
    } else {
      	
        if (value >= LONG_MIN && value <= LONG_MAX) {
          /*创建OBJ_STRING类型的robj*/
            o = createObject(OBJ_STRING, NULL);
          /* 如果待存储的值在LONG_MIN和LONG_MAX范围内，直接使用OBJ_ENCODING_INT作为编码格式*/
            o->encoding = OBJ_ENCODING_INT;
          /* 如果待存储的值在LONG_MIN和LONG_MAX范围内，直接将ptr指向该整数的地址*/
            o->ptr = (void*)((long)value);
        } else {
          //不在LONG_MIN和LONG_MAX范围时以SDS的方式存储该整数
            o = createObject(OBJ_STRING,sdsfromlonglong(value));
        }
    }
    return o;
}
```

​		如果string存储结构存储的数据是一个非纯数字字符串或者是一个大小不在lONG_MAX和LONG_MIN范围内的数字字符串，那么会使用sds的方式存储，并且当字符串字符个数少于OBJ_ENCODING_EMBSTR_SIZE_LIMIT(44)则使用OBJ_ENCODING_EMBSTR的方式，否则使用OBJ_ENCODING_RAW的方式：

```c
#define OBJ_ENCODING_EMBSTR_SIZE_LIMIT 44
robj *createStringObject(const char *ptr, size_t len) {
    if (len <= OBJ_ENCODING_EMBSTR_SIZE_LIMIT)
        return createEmbeddedStringObject(ptr,len);
    else
        return createRawStringObject(ptr,len);
}
```

​		从源代码中可以看到，在创建string对象时，会先检测ptr字符数组的长度是否超过OBJ_ENCODING_EMBSTR_SIZE_LIMIT的长度，OBJ_ENCODING_EMBSTR_SIZE_LIMIT的长度默认是44，如果未超过该长度时选择使用OBJ_ENCODING_EMBSTR格式的sds，否则使用OBJ_ENCODING_RAW的格式；

​		OBJ_ENCODING_RAW和OBJ_ENCODING_EMBSTR格式底层都是由sds实现的，但是创建过程有所不同；OBJ_ENCODING_EMBSTR的创建过程：

```c
robj *createEmbeddedStringObject(const char *ptr, size_t len) {
  	//同时分配redisObject和sdshdr8所需的空间
    robj *o = zmalloc(sizeof(robj)+sizeof(struct sdshdr8)+len+1);
    struct sdshdr8 *sh = (void*)(o+1);
		//初始化redisObject的成员
    o->type = OBJ_STRING; /*将type成员设置为OBJ_STRING*/
    o->encoding = OBJ_ENCODING_EMBSTR; /*将encoding成员设置为OBJ_ENCODING_EMBSTR*/
    o->ptr = sh+1; /*将ptr指向sdshdr结构体*/
    o->refcount = 1; /*将引用数+1*/
  	//检测是否需要执行淘汰流程
    if (server.maxmemory_policy & MAXMEMORY_FLAG_LFU) {
        o->lru = (LFUGetTimeInMinutes()<<8) | LFU_INIT_VAL;
    } else {
        o->lru = LRU_CLOCK();
    }
		//初始化sdshdr成员
    sh->len = len;
    sh->alloc = len;
    sh->flags = SDS_TYPE_8;
    if (ptr) {
        memcpy(sh->buf,ptr,len);
        sh->buf[len] = '\0';
    } else {
        memset(sh->buf,0,len+1);
    }
  //返回创建好的redisObject
    return o;
}

```

OBJ_ENCODING_RAW的创建过程：

```c
//创建Raw格式的string存储结构
robj *createRawStringObject(const char *ptr, size_t len) {
 		 /*Raw格式的string通过createObject来实现
  		*先通过sdsnewlen来创建sdshdr结构体，然后通过createObject创建redisObject结构体
  		*/
    return createObject(OBJ_STRING, sdsnewlen(ptr,len));
}
```

​		从源代码的实现即可发现：OBJ_ENCODING_RAW会分别分配redisObject和sdshdr结构体的空间，而OBJ_ENCODING_EMBSTR则会一并分配redisObject和sdshdr所需要的空间，因此OBJ_ENCODING_EMBSTR格式的创建和销毁过程都要比OBJ_ENCODING_RAW性能要高；

> **为何分界线是44？**

​		原因在于redisObject结构体大小占用16个字节，而sdshdr8占用3个字节，在创建sdshdr时还会加上一个'\0'来兼容c中的字符串，因此sdshdr总共占用4个字节，那么redisObject和sdshdr总共占用20个字节，而redis使用jealloc内存分配器，这种内存分配器仅能够分配8、16、32、64字节大小的内存，因此按照一次能够申请的最大内存来看，只剩余44个字节给存储字符串用，所以，redis张红将44作为row和embstr格式的分界线；

## 6.关于整数对象共享池:

​		redis在启动时，还会自动创建一批整数对象，即整数类型的redisObject，称之为整数对象共享池，用来实现整数对象的共享，以节省内存空间；这些整数对象在运行期间会一直存在，在向redis中存储一个数字类型的数据时，如果这个数字的范围在共享整数对象范围内，那么redis并不会再去创建这个整数对应的redisObject，而是将共享整数对象池中的整数对象引用计数+1；然后将该整数对象返回；这样的话在redis中大量存储整数类型数据时可以节省不少内存空间；

## 7.sds在使用中的优化：

​		sds在redis中创建好之后并非一成不变，在redis扩容之后，会根据扩容之后的长度重新分配sdshdr结构体，保证字符串始终以正确的格式存储；同时也会在设置一个String类型的数据或者向其他数据结构中添加一个字符串时对该字符串进行优化，来节省内存空间：具体如下：

​		1.检测该sds是否是ROW或者EMBSTR类型，如果是上述类型才能够进行优化，如果是INT类型，则直接返回；

​		2.检测该sds字符串长度是否小于22，以及是否能够转换为long型数据，如果长度小于22，并且能够转换为long型数据时将会转换为long型数据，然后使用INT类型存储；

​		3.在将sds转换为INT类型的编码之后，还会检测该INT类型的数据是否在共享整数范围内，如果在，那么将共享整数的引用计数+1，然后销毁原sds，并把共享整数的地址返回；

​		4.如果无法转换为INT类型时，还会继续检测当前编码是否是ROW类型，以及该ROW类型是否能够转换为；EMBSTR类型，如果能够转换则转换为EMBSTR类型；

​		5.如果类型本身已经是EMBSTR类型，那么还会继续检测当前字符数组剩余容量是否超过总容量的十分之一，如果超过，那么会将字符数组截取为当前字符串实际长度，以减少内存空间占用；	

