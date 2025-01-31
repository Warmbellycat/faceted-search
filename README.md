[![PHP Version](https://img.shields.io/badge/php-7.3%2B-blue.svg)](https://packagist.org/packages/k-samuel/faceted-search)
[![Total Downloads](https://img.shields.io/packagist/dt/k-samuel/faceted-search.svg?style=flat-square)](https://packagist.org/packages/k-samuel/faceted-search)
![Build and Test](https://github.com/k-samuel/faceted-search/workflows/Build%20and%20Test/badge.svg?branch=master&event=push)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/b9d174969c1b457fa8a6c3b753266698)](https://www.codacy.com/manual/kirill.a.egorov/faceted-search?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=k-samuel/faceted-search&amp;utm_campaign=Badge_Grade)
[![Codacy Badge](https://api.codacy.com/project/badge/Coverage/b9d174969c1b457fa8a6c3b753266698)](https://www.codacy.com/manual/kirill.a.egorov/faceted-search?utm_source=github.com&utm_medium=referral&utm_content=k-samuel/faceted-search&utm_campaign=Badge_Coverage)
# PHP Faceted search library

Simple and fast faceted search without external servers like ElasticSearch and others.

Easily handles 300,000 products with 10 properties. If you divide the indexes into product groups or categories,
then for a long time you will not need scaling and more serious tools. Especially in conjunction with
RoadRunner or Swoole.

The library is optimized for performance at the expense of RAM consumption.

[Changelog](./changelog.md)

## Install

`
composer require k-samuel/faceted-search
`

## Aggregates
The main advantage of the library is the quick and easy construction of aggregates.

Simply about aggregates.

<img align="left" width="100" vspace="4" hspace="4" src="https://github.com/k-samuel/faceted-search/blob/master/docs/filters.png">
We have selected a list of filters and received as a result a list of products suitable for these filters.

In the user interface, we need to display only the general types of filters for the selected products and the number
of products with a specific filter value (intersection).

When user select each new parameter in the filters, we need to calculate the list of available options and their number
for new results.

This is easy enough. Even if the goods have a different structure of properties.
```php
<?php
$filterData = $search->findAcceptableFiltersCount($filters);
```


## Notes

_* Create index for each product category or type and index only required fields._


Use database to keep frequently changing fields (price/quantity/etc) and facets for pre-filtering.

You can decrease the number of processed records by setting records list to search in.
For example: list of ProductId "in stock" to exclude not available products.

## Performance tests

Tests on sets of products with 10 attributes, search with filters by 3 fields.

Bench v2.0.0 PHP 8.1.0 + JIT + opcache (no xdebug extension)

| Items count     | Memory   | Find             | Get Filters (aggregates) | Sort by field| Results Found    |
|----------------:|---------:|-----------------:|-------------------------:|-------------:|-----------------:|
| 10,000          | ~6Mb     | ~0.0007 s.       | ~0.004 s.                | ~0.0005 s.   | 907              |
| 50,000          | ~40Mb    | ~0.002 s.        | ~0.014 s.                | ~0.001 s.    | 4550             |
| 100,000         | ~80Mb    | ~0.004 s.        | ~0.028 s.                | ~0.001 s.    | 8817             |
| 300,000         | ~189Mb   | ~0.011 s.        | ~0.104 s.                | ~0.006 s.    | 26891            |
| 1,000,000       | ~657Mb   | ~0.050 s.        | ~0.426 s.                | ~0.030 s.    | 90520            |

Bench v1.3.3 PHP 8.1.0 + JIT + opcache (no xdebug extension)

| Items count     | Memory   | Find             | Get Filters (aggregates) | Sort by field| Results Found    |
|----------------:|---------:|-----------------:|-------------------------:|-------------:|-----------------:|
| 10,000          | ~7Mb     | ~0.0007 s.       | ~0.004 s.                | ~0.0003 s.   | 907              |
| 50,000          | ~49Mb    | ~0.002 s.        | ~0.014 s.                | ~0.0009 s.   | 4550             |
| 100,000         | ~98Mb    | ~0.004 s.        | ~0.028 s.                | ~0.002 s.    | 8817             |
| 300,000         | ~242Mb   | ~0.012 s.        | ~0.112 s.                | ~0.007 s.    | 26891            |
| 1,000,000       | ~812Mb   | ~0.057 s.        | ~0.443 s.                | ~0.034 s.    | 90520            |

* Items count - Products in index
* Memory - RAM used for index
* Find - time of getting list of products filtered by 3 fields
* Get Filters - find acceptable filter values for found products.
  List of common properties and their values for found products (Aggregates)
* Sort by field - time of sorting found results by field value
* Results Found - count of found products (Find)

Experimental Golang port bench https://github.com/k-samuel/go-faceted-search

Bench v0.3.0 golang 1.17.3 with parallel aggregates

| Items count     | Memory   | Find             | Get Filters (aggregates) | Sort by field| Results Found    |
|----------------:|---------:|-----------------:|-------------------------:|-------------:|-----------------:|
| 10,000          | ~5Mb     | ~0.0004 s.       | ~0.001 s.                | ~0.0002 s.   | 907              |
| 50,000          | ~15Mb    | ~0.003 s.        | ~0.012 s.                | ~0.001 s.    | 4550             |
| 100,000         | ~21Mb    | ~0.006 s.        | ~0.028 s.                | ~0.003 s.    | 8817             |
| 300,000         | ~47Mb    | ~0.018 s.        | ~0.091 s.                | ~0.010 s.    | 26891            |
| 1,000,000       | ~150Mb   | ~0.094 s.        | ~0.408 s.                | ~0.040 s.    | 90520            |

## Examples

Create index using console/crontab etc.
```php
<?php
use KSamuel\FacetedSearch\Index;

$searchIndex = new Index();
/*
 * Getting products data from DB
 * Sort data by $recordId before using Index->addRecord it can improve performance 
 */
$data = [
    ['id'=>7, 'color'=>'black', 'price'=>100, 'sale'=>true, 'size'=>36],   
    ['id'=>9, 'color'=>'green', 'price'=>100, 'sale'=>true, 'size'=>40], 
    // ....
];
foreach($data as $item){ 
   $recordId = $item['id'];
   // no ned to add faceted index by id
   unset($item['id']);
   $searchIndex->addRecord($recordId, $item);
}
// save index data to some storage 
$indexData = $searchIndex->getData();
// We will use file for example
file_put_contents('./first-index.json', json_encode($indexData));
```

Using in application

```php
<?php
use KSamuel\FacetedSearch\Index;
use KSamuel\FacetedSearch\Search;
use KSamuel\FacetedSearch\Filter\ValueFilter;
use KSamuel\FacetedSearch\Filter\RangeFilter;
use KSamuel\FacetedSearch\Sorter\ByField;

// load index by product category (use request params)
$indexData = json_decode(file_get_contents('./first-index.json'), true);
$searchIndex = new Index();
$searchIndex->setData($indexData);
// create search instance
$search = new Search($searchIndex);
// get request params and create search filters
$filters = [
    new ValueFilter('color', ['black']),
    // RangeFilter example for numeric property ranges (min - max)
    new RangeFilter('size', ['min'=>36, 'max'=>40])
];
// Find records using filters. Note, it doesn't guarantee sorted result
$records = $search->find($filters);
// Now we have filtered list of record identifiers.
// Find records in your storage and send results into client application.

// Also we can send acceptable filters values for current selection.
// It can be used for updating client UI.
$filterData = $search->findAcceptableFilters($filters);

// If you want to get acceptable filters values with items count use findAcceptableFiltersCount
// note that filters is not applied for itself for counting
// values count of a particular field depends only on filters imposed on other fields 
$filterData = $search->findAcceptableFiltersCount($filters);


// If $filters is an empty array [], all acceptable values will be returned
$filterData = $search->findAcceptableFilters([]);

// Also you can sort results using FacetedIndex
$sorter = new ByField($searchIndex);
$records = $sorter->sort($records, 'price', ByField::SORT_DESC);




```

### Indexers

To speed up the search of RangeFilter by data with high variability of values, you can use the Range Indexer.
For example, a search on product price ranges. Prices can be divided into ranges with the desired step.

Note that RangeFilter is slow solution, it is better to avoid facets for highly variadic data

```php
<?php
use KSamuel\FacetedSearch\Index;
use KSamuel\FacetedSearch\Search;
use KSamuel\FacetedSearch\Indexer\Number\RangeIndexer;
use KSamuel\FacetedSearch\Filter\RangeFilter;

$index = new Index();
$rangeIndexer = new RangeIndexer(100);
$index->addIndexer('price', $rangeIndexer);

$index->addRecord(1,['price'=>90]);
$index->addRecord(2,['price'=>100]);
$index->addRecord(3,['price'=>150]);
$index->addRecord(4,['price'=>200]);


$filters = [
  new RangeFilter('price', ['min'=>100,'max'=>200])
];

$search = new Search($index);
$search->find($filters);

// will return [2,3]
```
RangeListIndexer allows you to use custom ranges list
```php
<?php
use KSamuel\FacetedSearch\Index;
use KSamuel\FacetedSearch\Indexer\Number\RangeListIndexer;

$index = new Index();
$rangeIndexer = new RangeListIndexer([100,500,1000]); // (0-99)[0],(100-499)[100],(500-999)[500],(1000 & >)[1000] 
$index->addIndexer('price', $rangeIndexer);
```
Also, you can create your own indexers with range detection method

### More Examples 
* [Demo](./examples)  
* [Performance test](./tests/performance/readme.md)


### Tested but discarded concepts

**Bitmap**
- (+) Significantly less RAM consumption
- (+) Comparable search speed with filtering
- (-) Limited by property variability, may not fit into the selected bit mask
- (-) Longer index creation
- (-) The need to rebuild the index after each addition of an element or at the final stage of formation
- (-) Slow aggregates creation

**Bloom Filter**
- (+) Very low memory consumption
- (-) Possibility of false positives
- (-) Slow index creation
- (-) Longer search in the list of products
- (-) Very slow aggregates

**Creating multiple indexes for different operations**
- (-) Increased consumption of RAM

# Q&A
* [Is it possible somehow to implement a full-text filter?](https://github.com/k-samuel/faceted-search/issues/3)
* [Would that be possible to use a DB as an index instead of a json file?](https://github.com/k-samuel/faceted-search/issues/5)