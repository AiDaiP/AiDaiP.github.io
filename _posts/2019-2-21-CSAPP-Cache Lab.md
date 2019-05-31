---
layout: post
title:  "CSAPP-Cache Lab"
date:   2019-2-21
desc: ""
keywords: ""
categories: [Cetus]
tags: [CSAPP]
icon: icon-html
---

# CSAPP-Cache Lab

* #### Part A 缓存模拟器

  实现一个缓存模拟器，读取给定的 trace 文件，输出对应的hit，miss，eviction次数

  

  * ##### cache的数据结构

    ![1](https://raw.githubusercontent.com/AiDaiP/images/master/cache%20lab/1.png)

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

  * ##### 判断是否命中

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

    

  * ##### 判断组是否已满

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

  * ##### load，store，modify

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

  * ##### 读取trace文件

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

  * ##### main函数

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

    ![2](https://raw.githubusercontent.com/AiDaiP/images/master/cache%20lab/2.png)

    gg，差1

    找了半天找不出来问题出在哪

    我选择死亡

    

* #### Part B 矩阵转制优化

  编写一个函数，计算给定矩阵的转置，最小化模拟缓存中的未命中数

  测试矩阵的尺寸为32\*32、64\*64、61\*67

  测试缓存 s = 5，E = 1，b = 5

  32组，一组1行，一个块32字节，存放8个int

  

  - ##### 矩阵转置

    ```c
    for (int i = 0; i < M; i++)
    {
        for (int j = 0; j < N; j++)
        {
            B[j][i] = A[i][j];
        }
    }
    ```

    

  - ##### 32 * 32

    ![3](https://raw.githubusercontent.com/AiDaiP/images/master/cache%20lab/3.png)

    矩阵一行32个int，数字代表数据储存的组，缓存32组，可以储存8\*32个int，8行填满一个cache，每隔8行出现冲突，造成冲突不命中

    把矩阵分成16个8\*8的小块，分别转置

    ```c
        if ( M == 32 && N == 32)
        {
            for (int i = 0; i < N; i += 8)
            {
                for (int j = 0; j < M; j += 8)
                {
                    for (int k = i; k < i + 8; k++)
                    {
                        int temp0 = A[k][j];
                        int temp1 = A[k][j+1];
                        int temp2 = A[k][j+2];
                        int temp3 = A[k][j+3];
                        int temp4 = A[k][j+4];
                        int temp5 = A[k][j+5];
                        int temp6 = A[k][j+6];
                        int temp7 = A[k][j+7];
                    
                        B[j][k] = temp0;
                        B[j+1][k] = temp1;
                        B[j+2][k] = temp2;
                        B[j+3][k] = temp3;
                        B[j+4][k] = temp4;
                        B[j+5][k] = temp5;
                        B[j+6][k] = temp6;
                        B[j+7][k] = temp7;
                    }
                }
            }       
        }
    ```

  - ##### 64*64

    ![5](https://raw.githubusercontent.com/AiDaiP/images/master/cache%20lab/5.png)

    一行64个int，4行填满一个cache，每隔4行出现冲突，造成冲突不命中

    分成4\*4的试试

    ```c
    if ( M == 64 && N == 64)
        {
        	for (int i = 0; i < N; i += 4)
            {
                for (int j = 0; j < M; j += 4)
                {
                    for (int k = i; k < i + 4; k++)
                    {
                        int temp0 = A[k][j];
                        int temp1 = A[k][j+1];
                        int temp2 = A[k][j+2];
                        int temp3 = A[k][j+3];
                    
                        B[j][k] = temp0;
                        B[j+1][k] = temp1;
                        B[j+2][k] = temp2;
                        B[j+3][k] = temp3;
                    }
                }
            }     
    	}
    ```

    ![6](https://raw.githubusercontent.com/AiDaiP/images/master/cache%20lab/6.png)

    miss应该在1300以下，1699，gg

    不可能用16\*16的，还是要用8\*8的搞，转置时还是要分成4\*4，俺寻思得把8\*8分成4个4\*4。

    然后就不会了

    记录一个对b操作，先转置再平移的做法

    ```c
    for (i = 0; i < N; i += 8) 
    {
            for (j = 0; j < M; j += 8) 
            {
                for (k = i; k < i + 4; k++) 
                {
                    a0 = A[k][j];
                    a1 = A[k][j + 1];
                    a2 = A[k][j + 2];
                    a3 = A[k][j + 3];
                    a4 = A[k][j + 4];
                    a5 = A[k][j + 5];
                    a6 = A[k][j + 6];
                    a7 = A[k][j + 7];
    
                    B[j][k] = a0;
                    B[j + 1][k] = a1;
                    B[j + 2][k] = a2;
                    B[j + 3][k] = a3;
    
                    B[j][k + 4] = a4;
                    B[j + 1][k + 4] = a5;
                    B[j + 2][k + 4] = a6;
                    B[j + 3][k + 4] = a7;
                }
                for (l = j + 4; l < j + 8; l++) 
                {
    
                    a4 = A[i + 4][l - 4]; // A left-down col
                    a5 = A[i + 5][l - 4];
                    a6 = A[i + 6][l - 4];
                    a7 = A[i + 7][l - 4];
    
                    a0 = B[l - 4][i + 4]; // B right-above line
                    a1 = B[l - 4][i + 5];
                    a2 = B[l - 4][i + 6];
                    a3 = B[l - 4][i + 7];
    
                    B[l - 4][i + 4] = a4; // set B right-above line 
                    B[l - 4][i + 5] = a5;
                    B[l - 4][i + 6] = a6;
                    B[l - 4][i + 7] = a7;
    
                    B[l][i] = a0;         // set B left-down col
                    B[l][i + 1] = a1;
                    B[l][i + 2] = a2;
                    B[l][i + 3] = a3;
    
                    B[l][i + 4] = A[i + 4][l];
                    B[l][i + 5] = A[i + 5][l];
                    B[l][i + 6] = A[i + 6][l];
                    B[l][i + 7] = A[i + 7][l];
                }
            }
        }
    ```

  - ##### 61\*67

    非对称矩阵，相差4行不一定冲突，直接分块

    试了一下8\*8，miss超出2000，16\*16可以

    ```c
    if ( M ==61 && N == 67)
        {
            for (int i = 0; i < N; i += 16)
            {
                for (int j = 0; j < M; j += 16)
                {
                    for (int k = i; k < i + 16 && k < 67; k++)
                    {
                    	for (int l = j; l < j + 16 && l < 61; l++)
                    	{
                    		  B[l][k] = A[k][l];
                    	}
                    }
                }
            }       	
        }
    ```

  ![7](https://raw.githubusercontent.com/AiDaiP/images/master/cache%20lab/7.png)

  