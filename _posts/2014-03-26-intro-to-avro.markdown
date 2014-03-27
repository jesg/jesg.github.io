---
layout: post
title:  "Introduction to Avro"
date:   2014-03-26 12:00:00
categories: hadoop avro
---

[Avro](https://avro.apache.org/) is a data serialization system.  Avro can be used to create compact, fast, binary format for hadoop.  Defining data types in avro is significantly easier than implementing the Writable interface in hadoop.

Avro defines data with schemas that are defined in json.  In this example we will define a Person with a Address in java and generate java classes with avro's maven plugin.


{% highlight json %}
{ 
	"type": "record",
	"name": "Person",
	"namespace": "jesg.avro",
	"doc": "A person has a name, phone number, and an address.",
	"fields": [
		{"name": "name", "type": "string"},
		{"name": "phonenumber", "type": "string", "order": "ignore"},
		{"name": "address", "type": 
			{ 
				"type": "record",
				"name": "Address",
				"namespace": "jesg.avro",
				"doc": "An Address has a street, state, city, and zip code",
				"fields": [
					{"name": "state", "type": "string"},
					{"name": "city", "type": "string"},
					{"name": "street", "type": "string", "order": "ignore"},
					{"name": "zipcode", "type": "string", "order": "ignore"}
				]
			}
		}
	]
}
{% endhighlight %}

This datatype will sort order will be by name, state, and then city.  We will configure the pom so the source files will be generated in the generate-sources phase.
{% highlight xml %}
...
  <properties>
  	<avro.version>1.7.5</avro.version>
  </properties>
  <dependencies>
	<dependency>
	  <groupId>org.apache.avro</groupId>
	  <artifactId>avro</artifactId>
	  <version>${avro.version}</version>
	</dependency>
  </dependencies>
  <build>
  	<plugins>
		<plugin>
		  <groupId>org.apache.avro</groupId>
		  <artifactId>avro-maven-plugin</artifactId>
		  <version>${avro.version}</version>
		  <executions>
		    <execution>
		      <phase>generate-sources</phase>
		      <goals>
		        <goal>schema</goal>
		      </goals>
		      <configuration>
		        <sourceDirectory>${project.basedir}/src/main/resources/avro</sourceDirectory>
		        <outputDirectory>${project.basedir}/src/main/java</outputDirectory>
		        <fieldVisibility>PRIVATE</fieldVisibility>
		      </configuration>
		    </execution>
		  </executions>
		</plugin>
  	</plugins>
  </build>
...
{% endhighlight %}

Now we need to serialize and deserialize a person in java.  First we will write a person to a file.

{% highlight java %}
Person person = Person.newBuilder()
        .setName("Bill")
        .setAddress(Address.newBuilder()
                .setState("TX")
                .setCity("Austin")
                .setStreet("42 Wishbone")
                .setZipcode("00000")
                .build())
        .setPhonenumber("000000000")
        .build();

DatumWriter<Person> writer = new SpecificDatumWriter<Person>(Person.class);
FileOutputStream out = new FileOutputStream("person-store");
Encoder encoder = EncoderFactory.get().binaryEncoder(out, null);
writer.write(person, encoder);
encoder.flush();
out.close();
{% endhighlight %}

Next we deserialize a person stored from a file.

{% highlight java %}
DatumReader<Person> reader = new SpecificDatumReader<Person>(Person.class);
FileInputStream in = new FileInputStream("person-store");
Decoder decoder = DecoderFactory.get().binaryDecoder(in, null);
Person result = reader.read(null, decoder);
System.out.println("Identity: " + (result == person) + " Equality: " + result.equals(person));
{% endhighlight %}

Code for this example is in [intro-avro](https://github.com/jesg/intro-avro).

