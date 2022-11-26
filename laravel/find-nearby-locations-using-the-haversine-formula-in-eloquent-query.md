---
description: >-
  Uses the haversine formula to find all DB records with a location within a
  specified radius
---

# Find nearby locations using the Haversine formula in Eloquent query

## Haversine

The **haversine formula** determines the [great-circle distance](https://en.wikipedia.org/wiki/Great-circle\_distance) between two points on a sphere given their [longitudes](https://en.wikipedia.org/wiki/Longitude) and [latitudes](https://en.wikipedia.org/wiki/Latitude). Important in navigation, it is a special case of a more general formula in spherical trigonometry, the **law of haversines**, that relates the sides and angles of spherical triangles. (Wikipedia)

### Managing coordinates

Personally, I have always found it easier to handle and store coordinates as strings. Very rarely is it required to perform any arithmetic operations on the values.

{% hint style="info" %}
If your requirements are more complex, and you have mysql 8, consider Spatial queries instead. [https://dev.mysql.com/doc/refman/8.0/en/spatial-types.html](https://dev.mysql.com/doc/refman/8.0/en/spatial-types.html)
{% endhint %}

### Eloquent or DB Query Builder

Add the following to an existing query builder instance, and then not forgetting to call `get()` or `paginate()`

The query assumes that your table contains columns called `latitude` and `longitude`. You may need to adapt these to suit your use case.

```sql
// the centre of your search
$latitude = '56.32124';
$longitude = '-1.342934';

// search radius
$distance = 5;  //(miles - see note)

$query->selectRaw('(3959 * acos (
       cos ( radians(?) )
       * cos( radians( latitude ) )
       * cos( radians( longitude ) - radians(?) )
       + sin ( radians(?) )
       * sin( radians( latitude )))) AS distance',[
           $latitude,
           $longitude,
           $latitude
  ]);
            
  $query->havingRaw('distance <= ? OR 0', [$distance]);
```

{% hint style="info" %}
&#x20;The above assumes that you have distance in miles.  If distance is in KM then replace 3959 with 6371.
{% endhint %}

References:

* [https://stackoverflow.com/questions/574691/mysql-great-circle-distance-haversine-formula](https://stackoverflow.com/questions/574691/mysql-great-circle-distance-haversine-formula)
* [https://en.wikipedia.org/wiki/Haversine\_formula](https://en.wikipedia.org/wiki/Haversine\_formula)
*

