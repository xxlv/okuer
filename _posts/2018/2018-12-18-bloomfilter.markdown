---
author: ghost
comments: true
date: 2018-11-27 08:05:39+00:00
layout: post
title: BloomFilter
categories:
- Tech
---

-------------------

#### 摘要

BloomFilter提供了一种判断一个值是否存在于指定集合的方法。在一个数据量较低的场景下，将数据保存到Hashset，然后利用hash检索某个值是否存在，但是在一个非常大的数据集中，传统的方法会消耗大量的内存，这导致OOM问题。BloomFilter的思路是，将一个value经过多次不同的hash映射到一个有限长度的bit数组中。当检测某个值是否存在的时候，只需要使用相同的hash检测,在bit数组中同时存在，则证明value大概率出现在集合中。如果有任何一次hash的index在bit map中被标记为0（代表无效），则证明value 一定不出现在该集合中。

之所以出现这个原因（大概率出现），是因为A和B两个value经过多次hash后，可能同时将bit map中相同的index标记为1。
使用bloomFilter可以节省大量的内存，在很多对结果要求并不很严格的场景下，bloomFilter是高效的。



-----
核心实现的代码如下


``` java

public class BloomFilter {

    private static final int MAX_HASH_COUNT=3;
    private byte[] map;
    private int size;

    public BloomFilter(int size){
        this.size=size;
        this.map=new byte[size];
        for (int i=0;i<map.length;i++){
            map[i]=0;
        }
    }

    public void add(String e){
        for (int i=1;i<=MAX_HASH_COUNT;i++) {
            int index=hashCodeGroup(e,i);
            map[index]=(byte)1;
        }
    }

    public boolean isExists(String e){

        int checkPoint=0;
        for (int i=1;i<=MAX_HASH_COUNT;i++) {
            if(map[hashCodeGroup(e,i)]==1){
                checkPoint++;
            }
        }
        return checkPoint==MAX_HASH_COUNT;
    }

    private int hashCodeGroup(String e,int loopIndex)
    {
        switch (loopIndex){
            case 1:
                return hashCode1(e)% this.size;
            case 2:
                return hashCode2(e)% this.size;
            case 3:
                return hashCode3(e)% this.size;
            default:
                return 0;
        }
    }

    private int hashCode1(String data){

        int hash = 0;
        int i;
        for (i = 0; i < data.length(); ++i) {
            hash = 33 * hash + data.charAt(i);
        }
        return Math.abs(hash);
    }

    private int hashCode2(String data){
        final int p = 16777619;
        int hash = (int) 2166136261L;
        for (int i = 0; i < data.length(); i++) {
            hash = (hash ^ data.charAt(i)) * p;
        }
        hash += hash << 13;
        hash ^= hash >> 7;
        hash += hash << 3;
        hash ^= hash >> 17;
        hash += hash << 5;
        return Math.abs(hash);

    }
    private int hashCode3(String key) {
        int hash, i;
        for (hash = 0, i = 0; i < key.length(); ++i) {
            hash += key.charAt(i);
            hash += (hash << 10);
            hash ^= (hash >> 6);
        }
        hash += (hash << 3);
        hash ^= (hash >> 11);
        hash += (hash << 15);
        return Math.abs(hash);
    }


}

```

BloomFilter 的准确率取决于size和hash次数，以及好的Hash函数选择。  在防止缓存击穿这类问题上，可以使用Bloomfilter。

