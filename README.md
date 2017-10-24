GeoIP-lite (implemented with promises)
==========

A native NodeJS API for the GeoLite data from MaxMind.

This product includes GeoLite data created by MaxMind, available from http://maxmind.com/

introduction
------------

MaxMind provides a set of data files for IP to Geo mapping along with opensource libraries to parse and lookup these data files.
One would typically write a wrapper around their C API to get access to this data in other languages (like JavaScript).

GeoIP-lite instead attempts to be a fully native JavaScript library.  A converter script converts the CSV files from MaxMind into
an internal binary format (note that this is different from the binary data format provided by MaxMind).  The geoip module uses this
binary file to lookup IP addresses and return the country, region and city that it maps to.

Both IPv4 and IPv6 addresses are supported, however since the GeoLite IPv6 database does not currently contain any city or region
information, city, region and postal code lookups are only supported for IPv4.

philosophy
----------

I was really aiming for a fast JavaScript native implementation for geomapping of IPs.  My prime motivator was the fact that it was
really hard to get libgeoip built for Mac OSX without using the library from MacPorts.

why geoip-lite
--------------

So why are we called geoip-lite?  `npm` already has a [geoip package](http://search.npmjs.org/#/geoip) which provides a JavaScript
binding around libgeoip from MaxMind.  The `geoip` package is fully featured and supports everything that the MaxMind APIs support,
however, it requires `libgeoip` to be installed on your system.

`geoip-lite` on the other hand is a fully JavaScript implementation.  It is not as fully featured as `geoip` however, by reducing its
scope, it is about 40% faster at doing lookups.  On average, an IP to Location lookup should take 20 microseconds on a Macbook Pro.
IPv4 addresses take about 6 microseconds, while IPv6 addresses take about 30 microseconds.

synopsis
--------

```javascript
var geoip = require('geoip-lite');

var ip = "207.97.227.239";

geoip.lookup(ip).then(function(geodata) {
  
  console.log(geodata);
  
    // { 
    //   range: [ 3479297920, 3479301339 ],
    //   country: 'US',
    //   region: 'TX',
    //   city: 'San Antonio',
    //   ll: [ 29.4889, -98.3987 ],
    //   metro: 641,
    //   zip: 78218
    // }
}).catch(function(err){
  console.log(err);
});

```

installation
------------
### 1. get the library

    $ npm install geoip-lite

### 2. update the datafiles (optional)

Run `npm run-script updatedb` to update the data files.

**NOTE** that this requires a lot of RAM.  It is known to fail on on a Digital Ocean or AWS micro instance.
There are no plans to change this.  `geoip-lite` stores all data in RAM in order to be fast.

API
---

geoip-lite is completely synchronous.  There are no callbacks involved.  All blocking file IO is done at startup time, so all runtime
calls are executed in-memory and are fast.  Startup may take up to 200ms while it reads into memory and indexes data files.

### Looking up an IP address ###

If you have an IP address in dotted quad notation, IPv6 colon notation, or a 32 bit unsigned integer (treated
as an IPv4 address), pass it to the `lookup` method.  Note that you should remove any `[` and `]` around an
IPv6 address before passing it to this method.

```javascript
geoip.lookup(ip).then(function(geodata) {
  console.log(geodata);
}).catch(function(err){
  console.log(err);
});;
```

If the IP address was found, the `lookup` method returns an object with the following structure:

```javascript
{
   range: [ <low bound of IP block>, <high bound of IP block> ],
   country: 'XX',                 // 2 letter ISO-3166-1 country code
   region: 'RR',                  // 2 character region code.  For US states this is the 2 letter
                                  // ISO-3166-2 subcountry code for other countries, this is the
                                  // FIPS 10-4 subcountry code
   city: "City Name",             // This is the full city name
   ll: [<latitude>, <longitude>], // The latitude and longitude of the city
   metro: <metro code>,           // Metro code
   zip: <postal code>             // Postal code (IPv4 only)
}
```

The actual values for the `range` array depend on whether the IP is IPv4 or IPv6 and should be
considered internal to `geoip-lite`.  To get a human readable format, pass them to `geoip.pretty()`

If the IP address was not found, the `lookup` returns `null`

### Pretty printing an IP address ###

If you have a 32 bit unsigned integer, or a number returned as part of the `range` array from the `lookup` method,
the `pretty` method can be used to turn it into a human readable string.

```javascript
    console.log("The IP is %s", geoip.pretty(ip));
```

This method returns a string if the input was in a format that `geoip-lite` can recognise, else it returns the
input itself.

### Start and stop watching for data updates ###

If you have a server running `geoip-lite`, and you want to update its geo data without a restart, you can enable
the data watcher to automatically refresh in-memory geo data when a file changes in the data directory.

```javascript
geoip.startWatchingDataUpdate();
```

This tool can be used with `npm run-script updatedb` to periodically update geo data on a running server.


Built-in Updater
----------------

This package contains an update script that can pull the files from MaxMind and handle the conversion from CSV.
A npm script alias has been setup to make this process easy. Please keep in mind this requires internet and MaxMind
rate limits that amount of downloads on their servers.

```shell
npm run-script updatedb
```

You can also run it by doing:

```bash
node ./node_modules/geoip-lite/scripts/updatedb.js
```

Or, if you really want, run the update once by `require('geoip-lite/scripts/updatedb.js')`.

Caveats
-------

This package includes the GeoLite database from MaxMind.  This database is not the most accurate database available,
however it is the best available for free.  You can use the commercial GeoIP database from MaxMind with better
accuracy by buying a license from MaxMind, and then using the conversion utility to convert it to a format that
geoip-lite understands.  You will need to use the `.csv` files from MaxMind for conversion.

Also note that on occassion, the library may take up to 5 seconds to load into memory.  This is largely dependent on
how busy your disk is at that time.  It can take as little as 200ms on a lightly loaded disk.  This is a one time
cost though, and you make it up at run time with very fast lookups.

References
----------
  - <a href="http://www.maxmind.com/app/iso3166">Documentation from MaxMind</a>
  - <a href="http://en.wikipedia.org/wiki/ISO_3166">ISO 3166 (1 & 2) codes</a>
  - <a href="http://en.wikipedia.org/wiki/List_of_FIPS_region_codes">FIPS region codes</a>

Copyright
---------

`geoip-lite` is Copyright 2011-2012 Philip Tellis <philip@bluesmoon.info> and the latest version of the code is
available at https://github.com/bluesmoon/node-geoip

License
-------

There are two licenses for the code and data.  See the [LICENSE](https://github.com/bluesmoon/node-geoip/blob/master/LICENSE) file for details.
