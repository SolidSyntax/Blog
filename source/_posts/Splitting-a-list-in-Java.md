title: Splitting a list in Java
tags:
  - Java
  - Collections
  - Recursion
date: 2016-02-06 10:25:51
---
How to split a list into multiple parts of the same size seems like a common question ([1](http://stackoverflow.com/questions/2895342/java-how-can-i-split-an-arraylist-in-multiple-small-arraylists),[2](http://stackoverflow.com/questions/1910236/how-can-i-split-an-arraylist-into-several-lists)). Most answers either point to adding an additional library such as [Guava](https://github.com/google/guava) or [Apache Commons](https://commons.apache.org/proper/commons-collections/javadocs/api-release/org/apache/commons/collections4/ListUtils.html).
 
As an experiment I've tried a differend approach. Splitting a list by using recursion.

<!-- more-->

{% codeblock lang:java %}
	public static <T> List<List<T>> split(List<T> source, Integer partitionSize) {
        if(partitionSize < 1){
        	throw new IllegalArgumentException("partitionSize must be at least 1");
        }
		
		if(source.isEmpty()){
            return new ArrayList<>();
        }

        int currentChunkSize = Math.min(partitionSize, source.size());
        List<T> head = source.subList(0, currentChunkSize);
        List<T> tail = source.subList(currentChunkSize, source.size());

        List<List<T>> result = split(tail, partitionSize);
        result.add(0, head);
        return result;
    }
{% endcodeblock %}

Testing the code seems to generate a successful output: 
{% codeblock lang:java %}
@Test
 public void splitTest() {
        List<String> source = Arrays.asList("1", "2", "3", "4", "5");
       
        List<List<String>> result = split(source, 3);

        assertEquals(Arrays.asList("1", "2", "3"), result.get(0));
        assertEquals(Arrays.asList("4", "5"), result.get(1));
    }
{% endcodeblock %}

However there is a **word of warning**! Java does not support [Tail recursion](https://en.wikipedia.org/wiki/Tail_call) optimization. So depending on your JVMs stacksize you may receive a StackOverflowError if you split a collection in more then 7000 parts.
