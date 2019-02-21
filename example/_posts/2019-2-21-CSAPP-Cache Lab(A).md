# CSAPP-Cache Lab(A)

* Part A 缓存模拟器

  实现一个缓存模拟器，读取给定的 trace 文件，输出对应的hit，miss，eviction次数

  

  * cache的数据结构

    ![1](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/cache%20lab/1.png)

    valid_bit是行的有效位，tag是标记位，lru表示最近最久未使用，lru越大，越久未使用，需要牺牲行时牺牲lru最大的一行

    ```c
    typedef struct 
    {
        int valid_bit;
        unsigned long tag;
        int lru;
    } Cache_line;
    
    typedef struct 
    {
        Cache_line* lines;
    } Cache_set;
    
    typedef struct 
    {
        int S;
        int E;
        Cache_set* sets;
    } Cache;
    
    ```

    根据S，E，b初始化cache

    ```c
    void init_cache(int S, int E, int b, Cache* cache) 
    {
        cache->S = S;
        cache->E = E;
        cache->sets = (Cache_set*) malloc(S * sizeof(Cache_set));
        for (int i = 0; i < S; i++) 
        {
            cache->sets[i].lines = (Cache_line*) malloc(E * sizeof(Cache_line));
            for (int j = 0; j < E; j++) 
            {
                cache->sets[i].lines[j].valid_bit = 0;
                cache->sets[i].lines[j].time = 0;
            }
        }
    }
    ```

  * 判断是否命中

    命中需要满足两个条件

    1. 设置了有效位
    2. cache地址的标记位与请求的地址的标记位相同

    ```c
    int get_hit_index(Cache *cache, int set_index, int tag) 
    {
        for (int i = 0; i < cache->E; i++) 
        {
            if (cache->sets[set_index].lines[i].valid_bit == 1 && cache->sets[set_index].lines[i].tag == tag) 
            {
                return i;
            }
        }
    
        return -1;
    }
    ```

    若命中则返回命中的索引，否则返回-1

    

  * 判断组是否已满

    遍历所有行，只要有一行有效位为0则该组未满。

    ```c
    int get_empty_index(Cache *cache, int set_index, int tag) 
    {
        for (int i = 0; i < cache->E; ++i) 
        {
            if (cache->sets[set_index].lines[i].valid_bit == 0) 
            {
                return i;
            }
        }
        return -1;
    }
    ```

    若满则返回-1，未满则返回该行索引

  * load，store，modify

    ```c
    void load(Cache *cache, int set_index, int tag, int verbose) 
    {
        int hit_index = get_hit_index(cache, set_index, tag);
        if (hit_index < 0) 
        {
            miss++;
            int empty_index = get_empty_index(cache, set_index, tag); 
            if (verbose) 
            {
                printf("miss ");
            }    
            if (empty_index < 0) 
            { 
                eviction++;
                if (verbose) 
                {
                    printf("eviction ");
                }
                //比较lru大小
                int max_index = 0;
                int max = cache->sets[set_index].lines[0].lru;
                for (int i = 1; i < cache->E; i++)
                {
                    if (cache->sets[set_index].lines[i].lru > max)
                    {
                        max_index = i;
                        max = cache->sets[set_index].lines[i].lru;
                    }
                }
                //牺牲行
                cache->sets[set_index].lines[max_index].valid_bit = 1;
                cache->sets[set_index].lines[max_index].tag = tag;
                cache->sets[set_index].lines[max_index].lru = 0;
            } 
            else 
            {
                for (int i = 0; i < cache->E; i++) 
                {
                //把空行的有效位设为1，标志位设置为请求地址的标记位，lru设为0，其他行的lru+1
                    if (i == empty_index) 
                    {
                        cache->sets[set_index].lines[i].valid_bit = 1;
                        cache->sets[set_index].lines[i].tag = tag;
                        cache->sets[set_index].lines[i].lru = 0;
                    } 
                    else 
                    {
                       cache->sets[set_index].lines[i].lru++;
                    }
                }
            }
        } 
        else 
        {                            
            hit++;
            if (verbose) 
            {
                printf("hit ");
            }
            for (int i = 0; i < cache->E; i++) 
            {
            //命中行的lru改为0，其他行的lru+1
                if (i == hit_index) 
                {
                    cache->sets[set_index].lines[i].lru = 0;
                }
                else 
                {
                    cache->sets[set_index].lines[i].lru++;
                }
            }
        }
    }
    //store与load相同
    void store(Cache *cache, int set_index, int tag, int verbose) 
    {
        load(cache, set_index, tag, verbose);
    }
    //modify相当于一次载入+一次储存
    void modify(Cache *cache, int set_index, int tag, int verbose) 
    {
        load(cache, set_index, tag, verbose);
        store(cache, set_index, tag, verbose);
    }
    ```

  * 读取trace文件

    ```c
    void read_file(int s, int E, int b, char* file, Cache* cache, int verbose) 
    {
        FILE *p = fopen(file, "r");
        char opt;                   
        unsigned long address;              
        int size;                       
    
        while (fscanf(p, " %c %lx,%d", &opt, &address, &size) > 0) 
        {
            if (opt == 'I') 
            {
                continue;
            } 
            else 
            {
                int tag = address >> (b + s);
                int set_index = (address >> b) & ((1 << s) - 1);
                if (verbose == 1) 
                {
                    printf("%c %lx,%d ", opt, address, size);
                }
                if (opt == 'S') 
                {
                    store(cache, set_index, tag, verbose);
                }
                if (opt == 'M') 
                {
                    modify(cache, set_index, tag, verbose);
                }
                if (opt == 'L') 
                {
                    load(cache, set_index, tag, verbose);
                }
                if (verbose == 1) 
                {
                    printf("\n");
                }
            }
        }
    
    }
    ```

  * main函数

    main函数需要读取命令参数，使用getopt实现

    ```c
    int main(int argc, char *argv[]) 
    {
        int s, S, E, b;
        char file[20];
        int verbose = 0;
        Cache cache;
        int ch;
        while ((ch = getopt(argc, argv, "vs:E:b:t:")) != -1) 
        {
            switch (ch) 
            {
                case 'v':
                    verbose = 1;
                    break;
                case 's':
                    s = atoi(optarg);
                    S = pow(2, s);
                    break;
                case 'E':
                    E = atoi(optarg);
                    break;
                case 'b':
                    b = atoi(optarg);
                    break;
                case 't':
                    strcpy(file, optarg);
                    break;
                default:
                    break;
            }
        }
        init_cache(S, E, b, &cache);                                
        read_file(s, E, b, file, &cache, verbose);       
        printSummary(hit, miss, eviction);
        return 0;
    }
    ```

    ```c
    #include "cachelab.h"
    #include <stdio.h>
    #include <stdlib.h>
    #include <getopt.h>
    #include <string.h>
    #include <unistd.h>
    #include <math.h>
    
    typedef struct 
    {
        int valid_bit;
        unsigned long tag;
        int lru;
    } Cache_line;
    
    typedef struct 
    {
        Cache_line* lines;
    } Cache_set;
    
    typedef struct 
    {
        int S;
        int E;
        Cache_set* sets;
    } Cache;
    
    int hit = 0;
    int miss = 0;
    int eviction = 0;
    
    void init_cache(int S, int E, int b, Cache* cache) 
    {
        cache->S = S;
        cache->E = E;
        cache->sets = (Cache_set*) malloc(S * sizeof(Cache_set));
        for (int i = 0; i < S; i++) 
        {
            cache->sets[i].lines = (Cache_line*) malloc(E * sizeof(Cache_line));
            for (int j = 0; j < E; j++) 
            {
                cache->sets[i].lines[j].valid_bit = 0;
                cache->sets[i].lines[j].lru = 0;
            }
        }
    }
    
    
    int get_hit_index(Cache *cache, int set_index, int tag) 
    {
        for (int i = 0; i < cache->E; i++) 
        {
            if (cache->sets[set_index].lines[i].valid_bit == 1 && cache->sets[set_index].lines[i].tag == tag) 
            {
                return i;
            }
        }
    
        return -1;
    }
    
    int get_empty_index(Cache *cache, int set_index, int tag) 
    {
        for (int i = 0; i < cache->E; ++i) 
        {
            if (cache->sets[set_index].lines[i].valid_bit == 0) 
            {
                return i;
            }
        }
        return -1;
    }
    
    void load(Cache *cache, int set_index, int tag, int verbose) 
    {
        int hit_index = get_hit_index(cache, set_index, tag);
        if (hit_index < 0) 
        {
            miss++;
            int empty_index = get_empty_index(cache, set_index, tag); 
            if (verbose) 
            {
                printf("miss ");
            }    
            if (empty_index < 0) 
            { 
                eviction++;
                if (verbose) 
                {
                    printf("eviction ");
                }
                int max_index = 0;
                int max = cache->sets[set_index].lines[0].lru;
                for (int i = 1; i < cache->E; i++)
                {
                    if (cache->sets[set_index].lines[i].lru > max)
                    {
                        max_index = i;
                        max = cache->sets[set_index].lines[i].lru;
                    }
                }
                cache->sets[set_index].lines[max_index].valid_bit = 1;
                cache->sets[set_index].lines[max_index].tag = tag;
                cache->sets[set_index].lines[max_index].lru = 0;
            } 
            else 
            {
                for (int i = 0; i < cache->E; i++) 
                {
                    if (i == empty_index) 
                    {
                        cache->sets[set_index].lines[i].valid_bit = 1;
                        cache->sets[set_index].lines[i].tag = tag;
                        cache->sets[set_index].lines[i].lru = 0;
                    } 
                    else 
                    {
                       cache->sets[set_index].lines[i].lru++;
                    }
                }
            }
        } 
        else 
        {                            
            hit++;
            if (verbose) 
            {
                printf("hit ");
            }
            for (int i = 0; i < cache->E; i++) 
            {
                if (i == hit_index) 
                {
                    cache->sets[set_index].lines[i].lru = 0;
                }
                else 
                {
                    cache->sets[set_index].lines[i].lru++;
                }
            }
        }
    }
    
    void store(Cache *cache, int set_index, int tag, int verbose) 
    {
        load(cache, set_index, tag, verbose);
    }
    
    void modify(Cache *cache, int set_index, int tag, int verbose) 
    {
        load(cache, set_index, tag, verbose);
        store(cache, set_index, tag, verbose);
    }
    
    void read_file(int s, int E, int b, char* file, Cache* cache, int verbose) 
    {
        FILE *p = fopen(file, "r");
        char opt;                   
        unsigned long address;              
        int size;                       
    
        while (fscanf(p, " %c %lx,%d", &opt, &address, &size) > 0) 
        {
            if (opt == 'I') 
            {
                continue;
            } 
            else 
            {
                int tag = address >> (b + s);
                int set_index = (address >> b) & ((1 << s) - 1);
                if (verbose == 1) 
                {
                    printf("%c %lx,%d ", opt, address, size);
                }
                if (opt == 'S') 
                {
                    store(cache, set_index, tag, verbose);
                }
                if (opt == 'M') 
                {
                    modify(cache, set_index, tag, verbose);
                }
                if (opt == 'L') 
                {
                    load(cache, set_index, tag, verbose);
                }
                if (verbose == 1) 
                {
                    printf("\n");
                }
            }
        }
    
    }
    
    
    int main(int argc, char *argv[]) 
    {
        int s, S, E, b;
        char file[20];
        int verbose = 0;
        Cache cache;
        int ch;
        while ((ch = getopt(argc, argv, "vs:E:b:t:")) != -1) 
        {
            switch (ch) 
            {
                case 'v':
                    verbose = 1;
                    break;
                case 's':
                    s = atoi(optarg);
                    S = pow(2, s);
                    break;
                case 'E':
                    E = atoi(optarg);
                    break;
                case 'b':
                    b = atoi(optarg);
                    break;
                case 't':
                    strcpy(file, optarg);
                    break;
                default:
                    break;
            }
        }
        init_cache(S, E, b, &cache);                                
        read_file(s, E, b, file, &cache, verbose);       
        printSummary(hit, miss, eviction);
        return 0;
    }
    
    ```

    ![2](https://raw.githubusercontent.com/AiDaiP/AiDaiP.github.io/master/images/cache%20lab/2.png)

    gg，差1

    找了半天找不出来问题出在哪

    我选择死亡

    