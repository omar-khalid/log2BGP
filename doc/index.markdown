---
layout: page
title: "BigPlg: A Open Source BGP Looking-glass"
date: 2013-11-02 06:45
comments: true
sharing: true
footer: true
---
### Big Protocol looking-glass: All your RIBs are belong to me
BigPlg allows the collection of the IPv4 Internet routing table which may be viewed by service providers and/or ISP customers to validate IP routing across autonomous systems.

Routing updates may also have attributes aliased to become (graph) readable. This function exists to allow the exporting of RIB updates to Splunk, or any such data collector.

A beneficial side-effect of collecting BGP routing information is the ability to track BGP updates/events local or remote(upstream). To analyze events Splunk is supported. A JSON/Socket API is currently under development.

Custom graphs can be easily implemented if taken the time to do so. There's certainly no need to rely on Splunk if one developed another way of displaying data. This project is meant to be nothing more than a route collector, and it's interface is solely CLI(command line interface). This project does not have graph data "out of the box".

![bgplg](http://aaronhebert.net/bgplg/images/screeny.png?raw=true)

### Enhancing Automation
On large-scale networks, this daemon may serve as a BGP routing information base for scripts to harvest network information. This eliminates the need to "screen scrape" output with regex, which always leaves me feeling dirty. A JSON/Socket API is currently under development to serve this purpose.

### Documentation
* **[Installing](http://aaronhebert.net/bgplg/installing)**
* **[Command-line Basics](http://aaronhebert.net/bgplg/start-cli)** 
* **[Command-line Refence](http://aaronhebert.net/bgplg/cli-reference)**
* **[JSON API Reference](http://aaronhebert.net/bgplg/api-reference)**

### Release Notes on BigPlg v1.0.3(captain's log)

[bigplg-1.0.3-GNU-Linux-x86_64.deb](http://aaronhebert.net/bgplg/release/bigplg-1.0.3-GNU-Linux-x86_64.deb "bigplg-1.0.3-GNU-Linux-x86_64.deb") - Nov. 25 2013

New capabilities in this revision, and 1 bug fix.
I've decided to use the daemons current binary database as a means of indexing BGP updates. These entries can be queried via a URL served by the ruby library Sinatra, or directly served via TCP socket API call.

The BGP event database is organized by [peer id|prefix] key pair in the RIB, and that RIB entry holds all indexes for BGP event data. Each prefix may occupy up to 1000 entries. Once the max entry limit is reached (1000 entries), the oldest entries will begin to be overwritten. It's recommended that you export to some log collector if you wish to keep the data.

#### Additions
* Created version definitions in `version.h`.
* Created TCP listener for Sinatra/socket API
* Converts all API call returns to JSON formatted strings.
* Supports wide(2byte) and narrow(1byte) character TCP sockets. e.g Unicode support
* Reaching the sinatra API:
	+ Current:
		+ http://[serverip]:4567/bgp/ipv4/:prefix
	+ History:
		+ http://[serverip]:4567/bgp/ipv4/:prefix/history
* Reaching the TCP socket level API:
	+ open TCP socket to [serverip] port 667
    	+ write command length followed by api-get
    	+ can be tested with `telnet localhost 667`

#### Socket API with Ruby example
``` ruby
    ## BGP Looking Glass Socket API example
    
    require 'socket'      # Sockets are in standard library
    
    def api_write(cmd)
    
      host = '127.0.0.1'
      port = 667
    
      s = TCPSocket.open(host, port)
      
      # Here we need to prepend the string length of the command.
      # The API will expect the length first.
      s.write(cmd.length.chr+cmd.to_s)
      
      # ret_val is JSON formatted after this read
      ret_val = s.read   # Read lines from the socket
      s.close            # Close the socket when done
      
      return ret_val.to_s
    end
    
    # Current prefix data
    puts("current data for 8.8.8.8 is:")
    puts(api_write("api-get prefix 8.8.8.8"))
    
    # Current prefix data, and historical
    puts("current & historical data for 8.8.8.8 is:")
    puts(api_write("api-get prefix 8.8.8.8 history"))
```

#### Revisions
* Peer state machine revised to only use 1 dedicated thread for reads, and the main BGP thread handles all queue processing. This has reduced the number of threads from 2 per peer to 1.
* Collapsed TCP server loops for CLI and API into a single method.
This will reduce code bloat.

#### Bug fixes
* bgpd failed to create it's withdraw database if the file didn't already exist. Failure to open it's database resulted in BGP event data not being collected.

### TODO (in progress)

* Add more functionality to the API from this point forward.
* Doxygen code comment clean up.
* Hosting Doxygen on projects webpage.
* Make max database entries configurable instead of hard-coded limit 1000.
* Finish [cx-lang](https://github.com/ahebert/Cx) project enough to use in configuring bigplgd.

### Road-map (up coming plans)
* IPv6 and multi-protocol extensions.
* BGP Additional Paths support. [Cisco Documentation](http://www.cisco.com/en/US/docs/ios-xml/ios/iproute_bgp/configuration/xe-3s/asr1000/irg-additional-paths.html)
* The CLI application responsible for configuring `bgpd` __will be deprecated at some point in the future__. CLI will be replaced with a configuration language that will allow start-up and run-time configurations via configuration scripts. Yep, CLI is going away.
* Web front end for pulling JSON from the API and displaying in a friendly UI.
* Considering graphing BGP event data for the web front end.

### Bug Reporting
See [issues](https://github.com/ahebert/flowlab_bgp_lg/issues) to report issues and check for updates on what is currently being worked.

### Contributing
If you wish to contribute, please fork the repo and contact <aaron.hebert@gmail.com> if you have any questions.