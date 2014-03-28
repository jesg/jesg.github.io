---
layout: post
title:  "Hadoop - Large Aggregation"
date:   2014-03-24 07:07:48
categories: hadoop java
---

Often an alogrithm needs to aggregate values.  Counting how many times a word occurs in a documents is one such alogrithm.  

Often the output from the mapper is huge. Let us say that each document has 1,000 words then we have 1,000 key value pairs for each document.  In some cases the hadoop cluster may not have enough memory the shuffle and  sort the values.

[Hadoop][hadoop] provides a built in solution called a combiner function.  A combiner functions input is the mappers output and forms the input for the reducer.  The combiner is an optimization that will be run an arbitrary number of times.

Another solution is to aggregate the values in the mapper.

{% highlight java %}
class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    private Map<String, Integer> partialResults;

    @Override
    protected void setup(Context context) throws IOException,
            InterruptedException {
        partialResults = new HashMap<String, Integer>();
    }

    @Override
    public void map(LongWritable key, Text value, Context context)
            throws IOException, InterruptedException {

        String line = value.toString();
        StringTokenizer tokenizer = new StringTokenizer(line);

        while (tokenizer.hasMoreTokens()) {
            increment(tokenizer.nextToken());
        }
    }

    private void increment(String word) {
        Integer count = partialResults.get(word);

        if (count == null) {
            count = 0;
        }

        count++;

        partialResults.put(word, count);
    }

    @Override
    protected void cleanup(Context context) throws IOException,
            InterruptedException {

        for (Entry<String, Integer> entry : partialResults.entrySet()) {
            context.write(new Text(entry.getKey()),
                    new IntWritable(entry.getValue()));
        }

    }
}
{% endhighlight %}

In this solution the aggregate values for the word count are stored in `partialResults`.  For very large `partialResults` we can flush `partialResults` after the map exceeds a specified size.

Code for this example is in the word-count module in [hadoop-patterns](https://github.com/jesg/hadoop-patterns).

[hadoop]: https://hadoop.apache.org/