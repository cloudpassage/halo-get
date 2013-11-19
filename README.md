halo-get
========

This script places a GET call to the Halo API, sending the API response to stdout.

Requirements:
* ruby
* The following ruby gems installed: oauth2, rest-client, json, and optparse.  The following command will install all optional gems needed by the CloudPasssage API clients:

```
sudo gem install oauth2 rest-client json public_suffix ip
```

* A Read only (preferred) or Full access API key and secret (*), placed in /etc/halo-api-keys separated by a vertical pipe, like:

```
aa00bb44|11111111222222223333333344444444
```

* This file should be owned by the user that runs api scripts, mode 600. Developers only: If you're working with an alternate grid, put that  grid's api hostname and port in the third column of the line:

```
aa00bb44|11111111222222223333333344444444|api.example.com:9999
```

_These can be found in the Portal under Settings, Site Administration, API Keys._

##Installation:

Copy halo-get.rb and wlslib.rb to the same directory in your path. 
wlslib.rb will also be found if you copy it to any directory in your
ruby library search path, which can be seen by running
```
echo 'puts $:' | irb
```

##Usage:

For command line help, see help text and parameters:

```
halo-get.rb -h
```

###Examples of GET calls

Return a list of server groups in single line json:
```
halo-get.rb -i aabbcc00 -u '/groups' | less
```

Return a list of groups in "pretty printed" (multiple line, indented) format:
```
halo-get.rb -i aabbcc00 -u '/groups' -p | less
```

_From the above output, find a group of interest and get it's 32 character hex ID (following     "id":   ).  For this example we'll use
0123456789abcdef0123456789abcdef:_

Return just a single group:
```
halo-get.rb -i aabbcc00 -u '/groups/0123456789abcdef0123456789abcdef' | less
```

Return a list of servers in "pretty printed" (multiple line, indented) format:
```
halo-get.rb -i aabbcc00 -u '/servers' -p | less
```

Return a list of servers and save the output to disk for later processing:
```
halo-get.rb -i aabbcc00 -u '/servers' >servers.json
```

*To see the different API calls available, see the "CloudPassage API Developer Guide", which is at https://support.cloudpassage.com/entries/23105662-CloudPassage-API-Developer-Guide as of the time of this writing.*

*This version of the script is able to place GET, PUT, POST, and DELETE calls.*


###Example of using PUT.  

To use this, you must have a "Full Access" API key; a Read Only API key will not be able to modify the zone. Since the PUT method requires a payload, that is provided on stdin.

Retrieve all current IP Zones
```
halo-get.rb -i aabbcc00 -u '/firewall_zones' -p | less
```

*From the above, find a zone you'd like to modify.  You'll need to get the "id" of that zone from the above output; we'll assume it's 99990000aaaabbbbccccddddeeeeffff.  Pull down the current zone contents:*

```
halo-get.rb -i aabbcc00 -u '/firewall_zones/99990000aaaabbbbccccddddeeeeffff' -p >current_zone.json
```

Edit current_zone.json and make any changes you'd like. Finally, upload the modified zone:
```
halo-get.rb -i aabbcc00 -m PUT -u '/firewall_zones/99990000aaaabbbbccccddddeeeeffff' -p <current_zone.json
```
###Example of using GET and PUT in a single pipeline

If you know exactly what changes to make, such as changing IP address 111.122.133.144 to IP address 50.51.52.53, you can even do the entire task as a single pipeline:

```
halo-get.rb -i aabbcc00 -u '/firewall_zones/99990000aaaabbbbccccddddeeeeffff' -p \
 | sed -e 's/111\.122\.133\.144/50.51.52.53/' \
 | halo-get.rb -i aabbcc00 -m PUT -u '/firewall_zones/99990000aaaabbbbccccddddeeeeffff' -p
```

###Example of using DELETE.  This also requires a Full Access API key.

Delete the zone we've been working with:
```
halo-get.rb -i aabbcc00 -m DELETE -u '/firewall_zones/99990000aaaabbbbccccddddeeeeffff' -p
```

###Example of using POST.  This also requires a Full Access API Key.

1. Create a new IP zone: First, create a file containing the json request.  The API
Programmer's Guide contains examples that can be edited.  For this
example we're using the exact json suggested in the Guide:

```
cat new_zone.json 
{
 "firewall_zone" : {
  "name" : "databases",
  "ip_address" : "10.10.10.1,10.10.10.2,10.10.10.3"
 }
}
```

1. Now create this zone:

```
halo-get.rb -i aabbcc00 -m POST -u '/firewall_zones' -p <new_zone.json
```


##Advanced uses:

If you manage more than one Halo Portal organization, you can place the same API request to all of them by specifying "-i {key}" #multiple times (with corresponding lines for all keys in /etc/halo-api-keys .  You'll get one response per line in the default single-line format, and the responses will be concatenated with no separator in the -p "pretty print" format.
```
halo-get.rb -i aabbcc00 -i 7890abcd -u '/groups' | less
```
