---
published: true
title: "Use the PostgreSQL JDBC Driver Directly with ColdFusion"
layout: post
tags: [postgresql,til]
---
For reasons irrelevant to this post, I wanted to run a query directly via the PostgreSQL JDBC Driver, bypassing `cfquery`. To be clear, up front, I do not recommend doing this and I don't know of any practical use case for it. But, because I couldn't find much online, I thought it worth documenting.
<!--more-->

## With Lucee

Here's the approach I took, using the latest stable version of Lucee CFML (5.3.7+48). First, create the connection by passing your database credentials and connection string to the driver:

```cfc
// properties for your database credentials
props = createObject( 'java', 'java.util.Properties' );
props.setProperty("user", "your_db_user");
props.setProperty("password", "your_db_password");

driver = createObject( 'java', 'org.postgresql.Driver' );

connectionString = "jdbc:postgresql://localhost:5432/database_name";
conn = driver.connect( connectionString, props );
```

You can then execute your query and assign its ResultSet to a variable:

```cfc
st = conn.createStatement();

sql = "SELECT * FROM my_awesome_table";

rs = st.ExecuteQuery(sql);

// Lucee enables you to dump/view the query result
writeDump( var='#rs#', abort='true' );
```

## With Adobe ColdFusion

I also tried this with Adobe ColdFusion 2021. The `org.postgresql.Driver` class does not appear to be available by default. To work around this, you'll need to [download the latest jar for the PostgreSQL Driver](https://jdbc.postgresql.org/download.html) and place it in your class path.[^1] You can then create the connection and execute the query as shown in the Lucee example.

Unlike Lucee, when you dump the ResultSet you're not able to see the records returned. You'll need to iterate over the ResultSet, as [explained here](https://www.softwaretestinghelp.com/jdbc-resultset-tutorial/). For a table with `id` and `name` columns, it might look something like this:

```cfc
while( rs.next() )
  {
    id = rs.getInt(1);
    name = rs.getString(2);
    writeoutput( id & ' ' & name );
  }
```

With all that said - I don't know any reason why you wouldn't be using `cfquery` or `queryExecute`.

____

[^1]: Adobe provides some [documentation about this process](https://helpx.adobe.com/coldfusion/developing-applications/using-web-elements-and-external-objects/integrating-jee-and-java-elements-in-cfml-applications/enhanced-java-integration-in-coldfusion.html).
