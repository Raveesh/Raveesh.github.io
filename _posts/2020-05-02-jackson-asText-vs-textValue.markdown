---
layout: post
title: Jackson asText vs textValue
date: 2020-05-02 18:38 
description: Understanding the difference between asText  and textValue
img: jacksonAsTextVsTextValue.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Jackson,Java]
---

Its such a simple thing but its enough to make your mind boggle. I recently spent some time working on a legacy code base, and as expected it followed the norm of having no test cases and no documentation. The bug that was reported was that when the UI form opens up and some fields which are supposed to be empty had the value *null* in them. 

## Debugging

As usual, the debugging process started and we started looking at the returned value from the database and what was getting saved in the database. The code was splattered with *if* and else conditions. One of the things that caught our attention was a series of  *if* statements written in the form of .

```java
if (jsonNode.get(Constants.CATEGORY) != null) {
        category = jsonNode.get(Constants.CATEGORY).textValue();
     }
```

We knew that these values were getting displayed as "null" in the UI, so it was time to write a quick unit test . 

```java
package com.sample;

import static org.junit.Assert.assertEquals;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.junit.Test;

public class JsonNodeTest {

  @Test
  public void testValues(){
    ObjectMapper mapper = new ObjectMapper();
    TestEntity fromValue = new TestEntity(2016, null);
    JsonNode node = mapper.valueToTree(fromValue);

    assertEquals(2016, node.get("id").intValue());
    assertEquals(null, node.get("name").textValue());

    //assertEquals(null, node.get("name").asText()); Throws assertion error
    assertEquals("null", node.get("name").asText());

  }


  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class TestEntity {
    private int id;
    private String name;
  }
}

```

The culprit was caught. While saving to the database, we were doing a conversion of *asText()* . The javadocs of *asText()* points out 

>Method that will return a valid String representation of the container value, if the node is a value node.

So *null* is returned as "null". (with the double quotes like a string)

JsonNode also exposes the *textValue()* method. This also returns the textual value if and only if it is a valid textual jsonNode. A null jsonNode is returned as null. The javadocs for *textvalue()* also state that 

>Does **NOT** do any conversions for non-String value nodes;for non-String values (ones for which {@link #isTextual} returns false) null will be returned.

We quickly changed a bunch of code which was calling the *asText()* method to call the *textValue()* method and voila! no more *null* on the UI. 
