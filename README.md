
# Working with XML


## Introduction
When you retrieve information from an API, it is often stored in either an XML or a JSON format. In this lesson, we look at what XML is, the problems it solves, and provide some hands on experience working with it. 

## Objectives
You will be able to:
* Explain what XML is and why it is used
* Retrieve information from an XML file and store it in a List or DataFrame

## eXtensible Markup Language

XML (eXtensible Markup Language) is a way of taking information and making it very easy for computers to read and process it. Here is an example:

```
<people>
    <person>
        <firstname>Jane</firstname>
        <lastname>Anderson</lastname>
        <phone type="cell">111-111-1111</phone>
    </person>
    <person>
        <firstname>Joe</firstname>
        <lastname>Sonos</lastname>
        <phone type="office">111-111-1111</phone>
    </person>
    <person>
        <firstname>Alison</firstname>
        <lastname>Demming</lastname>
        <phone type="home">111-111-1111</phone>
    </person>
</people>
```
You will see that there are a set of Elements denoted by words surrounded by angle brackets (they're called tags).

You'll also see that each Element starts with a tag and ends with the same tag but with a backslash - so a person starts with the `<person>` tag and ends with the `</person>` tag. You'll also notice that Elements can be nested - so the firstname Element is nested within each of the three person Elements.

Finally, you'll notice that in addition to Elements that have values (e.g. the first firstname element has a value of `Jane`), there are also attributes. In the example above the only attribute is a `type` attribute on the phone Element, allowing us to distinguish between home, work and cell phones. The difference between a value and an attribute is that a value is contained between the opening and closing tags - e.g. `<firstname>Alison</firstname>`, whereas an attribute is written *within* the opening tag for an element - e.g. `<phone type="office">`.

**Questions:**
* **What is the second Element for each of the people?**
* **Which is the only Element in the document with an attribute?**
* **What is the closing tag for the third Element within each of the `people` Elements?**


## The Problem it Solves - Structured, Hierarchical Data Description
It's important to understand the kinds of problems that XML *(or JSON - they both solve the same problem just using different syntax)* is optimized to solve.

### Best for Structured Data
Unstructured data is something like a book or an essay - lots of words, but free form text. While unstructured data can be contained within an XML document, the strength of XML documents is in describing structured information. Structured information is where the information is broken down into types of data. So for example, I could write an unstructured essay describing a person, but I could also create a structured document describing attributes of the person such as their first name, last name, date of birth and home phone number. Such a document could be a good use case for being stored in an XML file.

### Best for Hierarchical Data
Lets say we wanted to store structured information on a list of people with their first name, last name and phone number. How could we store that? We would probably do what we've done to date. Have a csv file with column names in the first row. That would be easy to read (for either a person or a computer) and would work perfectly. So for simple structured data (that fits neatly into a simple single sheet in a spreadsheet with one type of data per column), you probably don't need to use XML *(although sometimes such information is provided using XML anyway)*.

The real strength of XML is for hierarchical data. Imagine now that we have information about a list of people. But in addition to having their first name, last name and phone number, we also have a list of one or more addresses for each person. Now how would we store and share that information?

If we know that nobody has more than (say) three addresses, we could just add a bunch of extra columns to the csv file with names like address1_street, address1_city, address1_state, address1_zip, address2_street, address2_city and so on. But what happens if some people have 20 addresses and some have none? It's a pretty inefficient way of storing the information adding twenty sets of columns, and if you suddenly find that one of the people has 21 addresses, you need to change the structure of the file and anyone working with the file will have to rewrite their scripts to handle the extra fields for that twenty first address.

With XML, it's easy. Each person can have a collection of zero or more addresses, so we might take the example above and edit it to become:

```
<people>
    <person>
        <firstname>Jane</firstname>
        <lastname>Anderson</lastname>
        <phone type="cell">111-111-1111</phone>
        <addresses></addresses>
    </person>
    <person>
        <firstname>Joe</firstname>
        <lastname>Sonos</lastname>
        <phone type="office">111-111-1111</phone>
        <addresses>
            <address type="home">
                <street>27 Magnolia Steet</street>
                <city>Maplewood</city>
                <state>NJ</state>
            </address>
            <address type="work">
                <street>4 Main Street</street>
                <city>Montclair</city>
                <state>NJ</state>
            </address>
        </addresses>
    </person>
    <person>
        <firstname>Alison</firstname>
        <lastname>Demming</lastname>
        <phone type="home">111-111-1111</phone>
        <addresses>
            <address type="home">
                <street>27 Magnolia Steet</street>
                <city>Maplewood</city>
                <state>NJ</state>
            </address>
            <address type="work">
                <street>12 Main Street</street>
                <city>Montclair</city>
                <state>NJ</state>
            </address>
        </addresses>
    </person>
</people>
```

As you can see, the XML gracefully handles both Jane who doesn't have any addresses on file, and Joe and Alison who both have two. And Joe and Alison could easily have many more addresses without changing the structure of the XML file - we'd just add some more `address` Elements within the `addresses` element that contains the collection of addresses.


## Loading XML Data

The ability to parse and process XML is so important that there is a standard library built right into Python for working with XML files. The full docs for the XML library are available [here](https://docs.python.org/3.7/library/xml.html).

For this project we'll just be using a submodule within that library - ElementTree:
https://docs.python.org/3.7/library/xml.etree.elementtree.html#module-xml.etree.ElementTree

Lets start by working with a very simple file - the address data above. The first thing we need to do is to import the ElementTree code, parse the XML file and then store those results in a variable we can have a look through.


```python
import xml.etree.ElementTree as ET

tree = ET.parse('people.xml')
root = tree.getroot()
print(type(root))
```

    <class 'xml.etree.ElementTree.Element'>


OK, now lets go look at the elements within the root of the XML document. It is a collection that is iterable, so we can iterate over the elements within the document using the `for ... in` syntax we learned earlier for looping over collections. Lets start by printing out all of the top level elements, just to see what the content looks like:


```python
for child in root:
    print(child.tag, child.attrib)
```

    person {}
    person {}
    person {}


OK, that's a start. So we know the root (people) is comprised of a set of person tags. Now lets see if we can dig down into those tags to start to retrieve useful information.


```python
for person in root:
    print(person.tag, person.attrib)
    print('Elements:')
    for element in person:
        print('\t', element.tag, element.attrib, element.text)
    print('\n')
```

    person {}
    Elements:
    	 firstname {} Jane
    	 lastname {} Anderson
    	 phone {'type': 'cell'} 111-111-1111
    	 addresses {} None
    
    
    person {}
    Elements:
    	 firstname {} Joe
    	 lastname {} Sonos
    	 phone {'type': 'office'} 111-111-1111
    	 addresses {} None
    
    
    person {}
    Elements:
    	 firstname {} Alison
    	 lastname {} Demming
    	 phone {'type': 'home'} 111-111-1111
    	 addresses {} None
    
    


There are a few things going on in the code above. Firstly, the variable names we're using like person and element could be called anything else. The following code works equally well:


```python
for duck in root:
    print(duck.tag, duck.attrib)
    print('Elements:')
    for elephant in duck:
        print('\t', elephant.tag, elephant.attrib, elephant.text)
    print('\n')
```

    person {}
    Elements:
    	 firstname {} Jane
    	 lastname {} Anderson
    	 phone {'type': 'cell'} 111-111-1111
    	 addresses {} None
    
    
    person {}
    Elements:
    	 firstname {} Joe
    	 lastname {} Sonos
    	 phone {'type': 'office'} 111-111-1111
    	 addresses {} None
    
    
    person {}
    Elements:
    	 firstname {} Alison
    	 lastname {} Demming
    	 phone {'type': 'home'} 111-111-1111
    	 addresses {} None
    
    


All Python knows is that root contains a collection of things we can iterate over, and some of those in turn contain a collection of other things that we can iterate over. What we call them doesn't matter to the computer, although it's a good idea to use meaningful names for variables so you can remember what your code was supposed to do when you come back to it later.

Secondly, if you're wondering what the `print('\t', elephant.tag, elephant.attrib, elephant.text)` does, it prints each of the comma delimited items onto a single row, so it starts by printing a tab character to indent the output (`\t`), then it prints the name of the tag, any attribute(s) the tag might contain, and finally the contents (`text`) of the tag.

There is also a convenience method .iter() that allows you to iterate through all sub generations, regardless of depth.


```python
for element in root.iter():
    print(element.tag, element.attrib, element.text)
```

    people {} 
    
    person {} None
    firstname {} Jane
    lastname {} Anderson
    phone {'type': 'cell'} 111-111-1111
    addresses {} None
    person {} None
    firstname {} Joe
    lastname {} Sonos
    phone {'type': 'office'} 111-111-1111
    addresses {} None
    address {'type': 'home'} None
    street {} 27 Magnolia Steet
    city {} Maplewood
    state {} NJ
    address {'type': 'work'} None
    street {} 4 Main Street
    city {} Montclair
    state {} NJ
    person {} None
    firstname {} Alison
    lastname {} Demming
    phone {'type': 'home'} 111-111-1111
    addresses {} None
    address {'type': 'home'} None
    street {} 27 Magnolia Steet
    city {} Maplewood
    state {} NJ
    address {'type': 'work'} None
    street {} 12 Main Street
    city {} Montclair
    state {} NJ


OK, so let's try to do something useful. Let's retrieve the phone number for each of the people and put them all in a list . . .


```python
phone_numbers = []
for person in root:
    for element in person:
        if element.tag == "phone":
            phone_numbers.append(element.text)
print(phone_numbers)
```

    ['111-111-1111', '111-111-1111', '111-111-1111']


Great - now lets try to get a list of addresses - lets just take the cities and make a list of them first.


```python
cities = []
for person in root:
    for element in person:
        if element.tag == "addresses":
            for address in element:
                for item in address:
                    if item.tag == "city":
                        cities.append(item.text)
print(cities)
```

    ['Maplewood', 'Montclair', 'Maplewood', 'Montclair']


OK, now it's your turn! Start by iterating over the document and creating & printing a list of first names:

Great, now go in and create a list of the states that the people have an address in:

There are some other ways of working with XML files. In addition to just iterating over all of the elements under a given node, you can also find elements by name and retrieve their values by calling `.text`. So if you wanted a list of first names, you could do the following:


```python
first_names = []
for person in root:
    first_names.append(person.find('firstname').text)
print(first_names)
```

    ['Jane', 'Joe', 'Alison']


Now it's your turn! Create a list of full names (hint, you'll have to concatenate the first name, a space and then the last name for every full name):

OK, in practice, you're usually going to want to create a DataFrame from your XML file. Here is the code to create a DataFrame containing the first and last names for each person:


```python
dfcols = ['firstname', 'lastname']
df = pd.DataFrame(columns=dfcols)

for person in root:
    firstname = person.find('firstname').text
    lastname = person.find('lastname').text
    df = df.append(pd.Series([firstname, lastname], index=dfcols), ignore_index=True)
df.head ()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>firstname</th>
      <th>lastname</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Jane</td>
      <td>Anderson</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Joe</td>
      <td>Sonos</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Alison</td>
      <td>Demming</td>
    </tr>
  </tbody>
</table>
</div>



Great, now create a DataFrame that contains the first name, last name and phone for every person:

## Extra Credit (1)
Sometimes you need to perform more complex transformations to create the DataFrame you want. Write some code to create a DataFrame with one row per address, containing the first name, last name and all of the address information for each address *(anyone without an address simply won't show up in the DataFrame)*.


```python
dfcols = ['firstname', 'lastname', 'street', 'city', 'state']
addresses = pd.DataFrame(columns=dfcols)

for person in root:
    firstname = person.find('firstname').text
    lastname = person.find('lastname').text
    for address in person.find('addresses'):
        street = address.find('street').text
        city = address.find('city').text
        state = address.find('state').text
        addresses = addresses.append(pd.Series([firstname, lastname, street, city, state], index=dfcols), ignore_index=True)
addresses.head ()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>firstname</th>
      <th>lastname</th>
      <th>street</th>
      <th>city</th>
      <th>state</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Joe</td>
      <td>Sonos</td>
      <td>27 Magnolia Steet</td>
      <td>Maplewood</td>
      <td>NJ</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Joe</td>
      <td>Sonos</td>
      <td>4 Main Street</td>
      <td>Montclair</td>
      <td>NJ</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Alison</td>
      <td>Demming</td>
      <td>27 Magnolia Steet</td>
      <td>Maplewood</td>
      <td>NJ</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alison</td>
      <td>Demming</td>
      <td>12 Main Street</td>
      <td>Montclair</td>
      <td>NJ</td>
    </tr>
  </tbody>
</table>
</div>



## Extra Credit (2)
There is another file in this directory - `nyc_2001_campaign_finance.xml`. Open the file using Python, explore it using an iterator, and create a DataFrame containing the Candidate Name, Primary Pay, General Pay, Runoff Pay and Total Pay for each candidate.

## Summary

Congratulations! Now you've got some hands on experience working with importing XML files and pulling out the data you want into a DataFrame. Next up, we'll look at how to do the same with JSON files.
