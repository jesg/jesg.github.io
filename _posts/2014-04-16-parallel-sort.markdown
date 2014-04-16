---
layout: post
title:  "Parallel Sort"
date:   2014-04-16 12:00:00
categories: java
---

Sort an array in parallel with fork/join.

Fork/Join
---------

Fork/Join is a concurrency framework introduced in Java 7.  Fork/Join is designed for algorithms that recursively split the work.  Fork/Join let's idle worker threads steal work from other threads.

####Sorting Algorithm:

-------------------
1. Split SortTask until size of subtask is less than the threshold
2. Sort the subsection of the array
3. recursively merge subsections

---------------------

The `invokeAll` method forks the given tasks and returns to the current task when both of the subtasks have completed or thrown an exception.

{% highlight java %}
package jesg;

import java.util.Arrays;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;

public class ParallelSort {
    private ForkJoinPool pool;

    public ParallelSort(ForkJoinPool pool) {
        this.pool = pool;
    }

    public void sort(int[] ar) {
        pool
        .submit(new SortTask(ar, ar.length / pool.getParallelism(), 0,ar.length))
        .join();
    }

    private static class SortTask extends RecursiveAction {
        private final int[] ar;
        private final int[] temp;
        private final int threshold;
        private final int lo;
        private final int hi;

        private SortTask(int[] ar, int[] temp, int threshold, int lo, int hi) {
            this.ar = ar;
            this.lo = lo;
            this.hi = hi;
            this.temp = temp;
            this.threshold = threshold;
        }

        SortTask(int[] ar, int threshold, int lo, int hi) {
            this.ar = ar;
            this.lo = lo;
            this.hi = hi;
            this.temp = new int[ar.length];
            this.threshold = threshold;
        }

        @Override
        protected void compute() {
            if (hi - lo < threshold) {
                Arrays.sort(ar, lo, hi);
            } else {
                int mid = (lo + hi) >>> 1;
                invokeAll(new SortTask(ar, temp, threshold, lo, mid),
                        new SortTask(ar, temp, threshold, mid, hi));
                merge(mid);
            }

        }

        private void merge(final int mid) {
            int i = lo;
            int j = mid;
            int k = lo;

            while ((i < mid) && (j < hi)) {
                if (ar[i] <= ar[j]) {
                    temp[k++] = ar[i++];
                } else {
                    temp[k++] = ar[j++];
                }
            }

            while (i < mid)
                temp[k++] = ar[i++];

            while (j < hi)
                temp[k++] = ar[j++];

            for (int t = lo; t < hi; t++)
                ar[t] = temp[t];
        }
    }
}
{% endhighlight %}

Java 8
-----

In Java 8 arrays can be sorted in parallel with the `parallelSort` method in the Arrays class.  The sort runs on the common pool introduced in Java 8.

Benchmark
--------

The following benchmarks the sorting strategies in Java 8.

{% highlight java %}
package jesg;

import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.ForkJoinPool;

public class Benchmark {
    
    public static void main(String[] args) {
        ForkJoinPool pool = ForkJoinPool.commonPool();
        System.out.println("Parallelism: " + pool.getParallelism());
        
        int[] ar = new int[1000000];
        Random random = new Random();
        for(int i=0; i<ar.length; i++)
            ar[i] = random.nextInt();
        
        int reps = 50;
        
        long time = 0;
//        warm up threads
        for(int i=0; i<100; i++)
            jre8ParallelSort(Arrays.copyOf(ar, ar.length));
        
        for(int i=0; i<reps; i++)
            time += jre8ParallelSort(Arrays.copyOf(ar, ar.length));
        
        System.out.println("jre8 parallel: " + time/(double)reps);
        
        ParallelSort sorter = new ParallelSort(pool);
//        warm up threads
        for(int i=0; i<100; i++)
            forkJoin(Arrays.copyOf(ar, ar.length), sorter);
        
        time = 0;
        for(int i=0; i<reps; i++)
            time += forkJoin(Arrays.copyOf(ar, ar.length), sorter);

        System.out.println("fork/join: " + time/(double)reps);
        
        time = 0;
        for(int i=0; i<reps; i++)
            time += jre8Sort(Arrays.copyOf(ar, ar.length));
        
        System.out.println("jre8 single: " + time/(double)reps);
    }
    
    static long jre8ParallelSort(int[] ar){
        long start = System.currentTimeMillis();
        Arrays.parallelSort(ar);
        return System.currentTimeMillis() - start;
    }
    
    static long forkJoin(int[] ar, ParallelSort sorter){
        long start = System.currentTimeMillis();
        sorter.sort(ar);
        return System.currentTimeMillis() - start;
    }
    
    static long jre8Sort(int[] ar){
        long start = System.currentTimeMillis();
        Arrays.sort(ar);
        return System.currentTimeMillis() - start;
    }
}
{% endhighlight %}

On x86\_64 Linux 3.14 Oracle JDK 1.8.0

{% highlight text %}
Parallelism: 3
jre8 parallel: 57.44
fork/join: 55.6
jre8 single: 114.72
{% endhighlight %} 