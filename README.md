# MultiSourceDataDemo

## Getting the Code
The code for this example is hosted here on GitHub. You may be familiar with GitHub as being a great collaboration tool for version control. It also hosts a number of good code examples to draw inspriations from.

1. Download a ZIP file of the code base.
![Download File](https://i.imgur.com/Qgfe9bE.png)

2. Extract the ZIP file to your Working Directory.

## Setting up the Environment
For this example we will be using [Visual Studio Code](https://code.visualstudio.com/). We will be also using a number of extensions for _Visual Studo Code_ to aid in the development. These are:
* [Live Server by Ritwick Dey](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)
    * *Live Server* allows us to automatically refresh the browser when ever we save our work. This speeds up the development cycle slightly.
* [Beautify by HookyQR](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
    * *Beautify* is an extension that will format our HTML, JS, and CSS code to be consistently styled and easier to read.



1. Install _Visual Studio Code_ if it is not already installed on your machine.
2. Add _Live Server_ and _Beautify_ if they are not already installed in the editor.
3. Open the _extracted_ folder you just downloaded in the editor.
4. If everything is installed and the folder opened correctly, you should see a ***Go Live*** button in the bottom navigation bar of _Visual Studio Code_.

## Exploring the First Dataset
We are going to be using the _Queensland Government_'s [rental bond data](https://data.qld.gov.au/dataset/rental-bond-data) that has been lodged with the _Residential Tenancies Authority_. We will be using the ***March 2015*** dataset as an example however feel free to select another quarter to work with.

1. Select the ***RTA rental bond data—March 2015*** dataset.
2. Open the _Visualisation Preview_ and have a look at what the data looks like. You will see there is _a lot_ of records. The entire dataset is around 5MB. This will take a long time to load if we downloaded everything, everytime we open the page.

## Exercise 1: Using SQL to Filter API Results - Suburbs Only
As seen above the dataset is _large_. To make the page load faster we are going to store the dataset in the browser local storage. This however only works after the first load. To speed up the first load we are going to just get a small portion of the dataset to begin with. Following this we will only get parts of the dataset that the end user is interested in viewing. This will also be stored in the browser local storage.

### Prepare SQL satement to retrieve suburbs from dataset
To get the list of suburbs we have a few options:
1. Get the entire dataset, extract just the suburb column, and then remove duplicates.
2. Get the just the suburb column and then remove duplicates.
3. Get just a unique list of suburbs.

We are going to do the third option as it will be simplier for us to deal with and faster to process and store.

The data portal API supports using Structured Query Language (SQL) to query the datasets.

[W3 Schools SQL Tutorial](https://www.w3schools.com/sql/default.asp)  
[W3 Schools SQL - SELECT DISTINCT](https://www.w3schools.com/sql/sql_distinct.asp)

To get a unique list of items in a column we can use a `SELECT DISTICT` statement.

These are normally structured as  
```sql
SELECT DISTINCT column_name
FROM table_name
```
When working with the data portal API, the `column_name` is the name of the column you are after in the dataset while the `table_name` is the _resource id_ of the dataset. The API automatically converts `column_name` to lower case - for some datasets this will be a problem. This also is a problem if the column has a space. To counteract this we should wrap both `column_name` and `table_name` in double quotes (`"`).

So we have
```sql
SELECT DISTINCT "locality"
FROM "5edaa132-b4fd-4a47-84d1-44bc76e80c50"
```

To convert this to a URL which we can use to call the API we need to use _HTML URL Encoding_. This will replace special characters with a percentage encoding. We can use `encodeURI()` in Javascript to encode the URL _or we can just get a formatted URL to use_.

[W3 Schools HTML URL Encoding (Percent Encoding)](https://www.w3schools.com/tags/ref_urlencode.asp)

The base URL that we will use to call the API is
```
https://data.qld.gov.au/api/action/datastore_search_sql?sql=
```

Adding the SQL statement we get the full URI
```
https://data.qld.gov.au/api/action/datastore_search_sql?sql=SELECT%20DISTINCT%20%22locality%22%20FROM%20%225edaa132-b4fd-4a47-84d1-44bc76e80c50%22
```
Visit this and check out the response. You should be able to see a large JSON response of just suburbs.

Now we need to add this into our code and get it pulling.

### AJAX call to get data
If you haven't already done so, open up `script.js` file and have a look. You will see a number of sections that are marked as ***TODO***. The first section we will do is 

```javascript
/*
    TODO: Retrieve just a list of suburbs from the dataset and clean it up.
    [ ] Prepare SQL satement to retrieve suburbs from dataset
    [ ] AJAX call to get data
    [ ] Success method to process and store in local storage
*/
```

For the AJAX call, we are going to do it if the browser local storage does not contain the item *Suburbs*. The following is the base AJAX method. Add in the URL and then we will work on the success method.  

```javascript
$.ajax({
    type: "GET",
    url: "INSERT_URL_HERE",
    success: function (data) {

    },
    dataType: "text"
});
```

### Success method to process and store in local storage
The data that is returned is JSON however we are returning it as text. We can parse it to a JSON object by using the `JSON.parse()` function. We also want to get just the suburbs from the dataset, rather than the full response.
```javascript
        var jsonBlock = JSON.parse(data);
        var localities = jsonBlock.result.records;
```
If we look at the data we can see that all the suburbs are UPPERCASE. We can use the suburbs like this however to make it easier to read we want to change the suburbs to Title Case. For this we will convert the suburb entirely to lowercase than make the first letter of each word uppercase. The following code will do this. It also will sort the suburbs into alphabetical order. This will make it easier for our next exercise - autocomplete on filter inputs.
```javascript
        localities.forEach(function(item, index) {
            var suburb = item.locality;

            suburb = suburb.toLowerCase().split(' ').map(function(word) {
                return word.replace(word[0], word[0].toUpperCase());
            }).join(' ');

            cleanLocalities.push(suburb);
        });
        cleanLocalities.sort();
```
Lastly on this exercise, we need to store the cleaned up suburb list into local storage so we don't need to query the API everytime.
```javascript
        var jsonStorage = JSON.stringify(cleanLocalities);
        localStorage.setItem("Suburbs", jsonStorage);
```
To make sure the code is working as intended check the local storage section of the developer console. You make like to delete the local storage and refresh to see if it is loading correctly.

## Exercise 2: Autocomplete on Filter Inputs
To make it easier for the end user to use, we would like to add an autocomplete to the suburb search fields. This will allow us to start typing a suburb and we will be offered suggestions on how to complete it.

We are going to use [typeahead.js](https://github.com/twitter/typeahead.js). 

> Inspired by twitter.com's autocomplete search functionality, typeahead.js is a flexible JavaScript library that provides a strong foundation for building robust typeaheads.

>The typeahead.js library consists of 2 components: the suggestion engine, [Bloodhound](https://github.com/twitter/typeahead.js/blob/master/doc/bloodhound.md), and the UI view, [Typeahead](https://github.com/twitter/typeahead.js/blob/master/doc/jquery_typeahead.md). The suggestion engine is responsible for computing suggestions for a given query. The UI view is responsible for rendering suggestions and handling DOM interactions. Both components can be used separately, but when used together, they can provide a rich typeahead experience.

### Acquire the library and set it up for use

We are going to be using both _Bloodhound_ and _Typeahead_. For this we want to download [***typeahead.bundle.min.js***](http://twitter.github.com/typeahead.js/releases/latest/typeahead.bundle.min.js). Once you have downloaded this, place it in the `js` folder, and add the `<script>` tag to use the library.

Back in our `script.js` we are now moving on to our next ***TODOs***.

```javascript
/*
    TODO: Initalise autocomplete
    [ ] Call Bloodhound Typeahead Initaliser
*/
...
/*
    TODO: Function to initalise the autocomplete functionality
    [ ] Construct the engine behind the autocomplete
    [ ] Attached engine to input fields
    [ ] Set display options for the autocomplete
*/
```

### Construct the engine behind the autocomplete
We want to construct the engine as part of our `initaliseBloodhoundTypeAhead()` function. In this we need to setup some options. The options we need to set are:
>* datumTokenizer – A function with the signature (datum) that transforms a datum into an array of string tokens. **Required**.
>* queryTokenizer – A function with the signature (query) that transforms a query into an array of string tokens. **Required**.
>* local – An array of data or a function that returns an array of data. The data will be added to the internal search index when `#initialize` is called.

The default for the _tokenizers_ is `Bloodhound.tokenizers.whitespace` which we will use as is.
```javascript
function initaliseBloodhoundTypeAhead(suburbs) {
    var suburbSuggestions = new Bloodhound({
        datumTokenizer: Bloodhound.tokenizers.whitespace,
        queryTokenizer: Bloodhound.tokenizers.whitespace,
        local: suburbs
    });
}
```

### Attached engine to input fields
As part of this function, we also need to attached the engine to our two input fields. Looking at the HTML we can see that we can target both fields using `.suburb-select input`. To attached the engine, we call the `typeahead()` function on these elements.

```javascript
    $(".suburb-select input").typeahead();
```

### Set display options for the autocomplete
As part of the call to `typeahead()` we need to provide options and datasets for determining how the engine will run and what will be displayed.
The options we want to provide are:
>* `highlight` – If `true`, when suggestions are rendered, pattern matches for the current query in text nodes will be wrapped in a `strong` element with its class set to `{{classNames.highlight}}`. Defaults to `false`.
>* `hint` – If `false`, the typeahead will not show a hint. Defaults to `true`.
>* `minLength` – The minimum character length needed before suggestions start getting rendered. Defaults to `1`.

We also want to provide the following dataset options:
>* `source` – The backing data source for suggestions. Expected to be a function with the signature `(query, syncResults, asyncResults)`. `syncResults` should be called with suggestions computed synchronously and `asyncResults` should be called with suggestions computed asynchronously (e.g. suggestions that come for an AJAX request). `source` can also be a Bloodhound instance. **Required**.
>* `name` – The name of the dataset. This will be appended to `{{classNames.dataset}}-` to form the class name of the containing DOM element. Must only consist of underscores, dashes, letters (`a-z`), and numbers. Defaults to a random number.
>* `limit` – The max number of suggestions to be displayed. Defaults to `5`.

```javascript
    $(".suburb-select input").typeahead({
        hint: true,
        highlight: true,
        minLength: 1
    },
    {
        name: 'suburbs',
        source: suburbSuggestions,
        limit: 10
    });
```

### Call Bloodhound Typeahead Initaliser
Lastly for this exercise we need to call the initaliser once we have the list of suburbs. There are two place we need to do this, once when the AJAX returns and the other if we use the local storage suburbs.

```javascript
    initaliseBloodhoundTypeAhead(cleanLocalities);
```

Once you have added this, go and test out the autocomplete.

## Exercise 3: Visualising Data with Google Charts

## Exercise 4: Using SQL to Retrieve a Matching Suburb Image
